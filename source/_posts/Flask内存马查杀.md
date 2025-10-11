---
title: Flask内存马查杀
toc: true
categories:
  - 技术
  - web
date: 2025-05-28 23:10:12
tags:
  - python
  - web安全
---

本文介绍如何在 Flask 应用的内存中查杀内存马（Webshell），重点是如何通过操作 Flask 应用对象 `<Flask 'app'>` 来发现并清除恶意代码。

------

内存马通常是通过动态代码注入的方式存在于内存中，利用 Flask 应用的生命周期和钩子函数来隐藏自身行为。查杀内存马关键在于：

- 找到 Flask 进程对应的 Python 运行环境
- 访问 Flask 应用对象，分析其属性和注册的钩子、路由等
- 识别并定位恶意注入的函数或钩子，进行清除或禁用

------

## 环境准备

这里我们使用 **pyrasite** 这个开源工具，能够注入代码到运行中的 Python 进程，实时交互并调试。

### 安装 pyrasite

> 建议直接从 GitHub 克隆最新版安装，pypi的不行

```bash
git clone https://github.com/lmacken/pyrasite.git
pip install ./pyrasite
```

------

## 查杀步骤

### 1. 找到 Flask 进程 ID

使用命令查看运行中的 Flask 应用：

```bash
ps aux | grep python
```

或者针对 Flask 启动脚本查找：

```bash
ps aux | grep app.py
```

假设 Flask 进程 PID 为 `8`。

------

### 2. 注入交互 Shell

注入 Python 交互式 shell 到 Flask 进程：

```bash
pyrasite-shell 8
```

这样就可以在 Flask 进程中执行任意 Python 代码。

------

### 3. 访问 Flask 应用对象

进入 shell 后，找到 Flask 应用对象。一般常见变量名是 `app` 或 `application`，如果不知道变量名，可通过 `globals()` 或 `locals()` 查找。

常用属性包括：

```python
app.after_request_funcs       # 请求后钩子函数字典
app.before_request_funcs      # 请求前钩子函数字典
app.url_map                   # 路由映射表
app.view_functions            # 路由对应的处理函数字典
app.error_handler_spec        # 错误处理函数映射
```

------

### 4. 查找恶意钩子或路由

查看所有注册的路由及对应函数：

```python
for route, func in app.view_functions.items():
    print(route, func)
```

------

### 5. 分析错误处理钩子

```python
exc_class, code = app._get_exc_class_and_code(404)
malicious_handler = app.error_handler_spec.get(None, {}).get(code, {}).get(exc_class)#exc_class,code=app._get_exc_class_and_code(404);app.error_handler_spec[None][code][exc_class]
import dis
dis.dis(malicious_handler)
```

------

### 6. 反汇编可疑函数

利用 Python 的 `dis` 模块对可疑函数进行反汇编，查看底层字节码逻辑：

```python
import dis
dis.dis(app.view_functions['shell'])
```

通过字节码分析判断函数是否含有执行系统命令、执行动态代码等行为。
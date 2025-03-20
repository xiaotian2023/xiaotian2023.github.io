---
typora-root-url: ./java本地命令执行
title: java本地命令执行
toc: true
categories:
  - 技术
  - ctf
  - web
date: 2025-03-20 21:11:21
tags:
---

# java本地命令执行

### Runtime

![](image.png)

就一个构造方法，是private的，只能在Runtime类中实例化，但是可以使用静态方法getRuntime获取到Runtime对象，通过Runtime对象的方法exec执行命令

exec返回一个Process对象

调用链（windows）

```java
Runtime.exec() 
	->ProcessBuilder.start()
		-> ProcessImpl.start() 
     		-> new ProcessImpl()
    //都在java.lang下
```

linux是将ProcessImpl的start换成forkAndExec

Runtime.exec直接执行，或在jdk8以下反射执行

```java
System.out.println(org.apache.commons.io.IOUtils.toString(Runtime.getRuntime().exec("ls").getInputStream()));
//需要添加库org.apache.commons.io
```

### ProcessBuilder

类跟方法start都是public的，直接用，也是返回Process对象

```java
System.out.println(org.apache.commons.io.IOUtils.toString(new ProcessBuilder("whoami").start().getInputStream()));
```

### ProcessImpl

类是default权限，只能在java8以下反射获取，并调用static的start

```java
Class<?> clazz = Class.forName("java.lang.ProcessImpl");
String[] cmds = new String[]{"whoami"};
Method method = clazz.getDeclaredMethod("start", String[].class, Map.class, String.class, ProcessBuilder.Redirect[].class, boolean.class);
method.setAccessible(true);
Process e = (Process) method.invoke(null, cmds, null, null, null, false);
System.out.println(org.apache.commons.io.IOUtils.toString(e.getInputStream()));
```

### Jni命令执行

java中native方法是底层c/c++实现的（jni就是java与c/c++地层交流的api）

自己编写native方法测试

创建test2.java

java测试

```java
package test;
public class test2 {
    public static native String nativeMethod(String str);
    public static void main(String[] args) throws Exception {
        System.load("D:\\download\\untitled1\\test\\cmd.dll"); 	//你最后的dll生成的位置，或者你直接将dll丢掉java.library.path
        System.out.println(test2.nativeMethod("ls / -al"));
    }
}
```

```bash
javac -cp . test2.java -h .  //-h选项用来在指定目录生成test_test2.h
```

编写test_test2.cpp

java与c/c++通信参考（https://blog.csdn.net/qq_25722767/article/details/52557235）

```java
//
// Created by tiand on 2025-03-20.
//

#include "test_test2.h"
#include <string> 
#include <cstdio>
using namespace std;

JNIEXPORT jstring JNICALL Java_test_test2_nativeMethod(JNIEnv *env, jclass jclass, jstring str) {
    if (str == NULL) {
        return NULL;
    }

    const char *c_str = env->GetStringUTFChars(str, NULL);
    if (c_str == NULL) {
        return NULL;
    }

    FILE *fp = popen(c_str, "r");
    env->ReleaseStringUTFChars(str, c_str);  // 释放传入的字符串

    if (fp == NULL) {
        return NULL;
    }

    string output;  // 定义 string 存储完整的输出
    char buffer[1024];

    while (fgets(buffer, sizeof(buffer), fp) != NULL) {
        output += buffer;
    }
    pclose(fp);

    return env->NewStringUTF(output.c_str());  // 返回完整的多行字符串
}
```

编译命令

```cmd
g++ -shared -o cmd.dll test_test2.cpp -I"%JAVA_HOME%/include" -I"%JAVA_HOME%/include/win32" -static -static-libgcc -static-libstdc++ -Wl,--add-stdcall-alias

#-static尽量静态编译
```


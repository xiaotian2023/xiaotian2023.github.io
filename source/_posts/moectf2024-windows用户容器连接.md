---
title: moectf2024-windows用户容器连接
toc: true
categories:
  - 软件操作
date: 2024-09-04 22:25:29
tags:
  - 操作指导
---

​	关于很多人使用wsl和虚拟机连接容器出现的问题做一个简单指导，本文仅指导wsl以及虚拟机的连接方式

1. 开启容器并粘贴wsrx链接![](image1.png)一定一定别复制旁边的127.0.0.1:xxxx去访问
2. 修改wsrx配置![](image2.png)
3. 粘贴到wsrx中并点击右边箭头
4. 打开连接情况查看![](image3.png)有一个0.0.0.0:xxxx而不用127.0.0.1:xxxx，记录:后的端口
5. 查看你的虚拟机网络模式是否是nat模式![](image4.png)不是请修改为nat模式，不过一般默认是nat模式
6. 查看并关闭windows下公用网络防火墙![](image5.png)
7. 在你的windows终端中输入ipconfig查看ip![](image6.png)记住这两个的任意一个（wsl就不要记录VMnet8那个了)
8. 在虚拟机中访问记录的ip:记录的端口，例如我的 192.168.89.1:60785（wsl不能用这个）![](image7.png)或10.x.x.x:60785![](image8.png)，pwn题的话就可nc连接，比如 nc ip port

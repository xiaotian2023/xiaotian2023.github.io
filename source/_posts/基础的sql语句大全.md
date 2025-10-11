---
title: 基础的sql语句大全
toc: true
categories:
  - 技术
  - web
date: 2025-06-14 21:10:12
tags:
  - sqli
  - web安全
---

本篇为基础的sql语句，方便sql注入时忘记语法来进行查询

## CREATE/DROP/ALTER语句

### 模式（schema）

mysql中schema = database

```sql
-- 定义SCHEMA
create schema 模式名 authorization 用户名;
create schema authorization 用户名; -- 未定义模式名，默认模式名为用户名

-- 删除SCHEMA
drop schema 模式名 cascade; -- 级联
drop schema 模式名 restrict; -- 限制
```

### 基本表（table）

#### 定义基本表

```sql
#定义基本表
create table 表名
( 列名1 数据类型 列级完整性约束条件, -- 列级完整性约束条件可忽略
  列名n 数据类型 列级完整性约束条件,
  表级完整性约束条件1,
  表级完整性约束条件n
);

-- 数据类型
char(n)      -- 定长字符
varchar(n)   -- 最大长度n变长字符
number(n)    -- 数字型
int          -- 长整型4B
smallint     -- 短整型2B
bigint       -- 大整型8B
float(n)     -- 大约有效数字n
date         -- 日期，YYYY-MM-DD
time         -- 时间，HH:MM:SS

-- 列级完整性约束条件
primary key
not null
unique
check

-- 表级完整性约束条件
primary key(列名1, 列名n)
foreign key(列名1) references 被参照表(列名1)

#在模式中定义表
create table 模式名.表名
( 列定义语句,
  表完整性约束
);
-- Postgresql特殊语法
create schema 模式名 authorization 用户名
 create table 表名
 ( 列定义语句,
   表完整性约束
 );
ALTER TABLE 模式名.表名 SET SCHEMA 新模式名;
```

#### 修改基本表

``` sql
alter table 表名 add column 新列名 数据类型 完整性约束;

alter table 表名 add 列级完整性约束; -- add unique(列名)
alter table 表名 add 表级完整性约束;

alter table 表名 drop 列名 cascade; -- 级联
alter table 表名 drop 列名 restrict;

alter table 表名 drop constraing 完整性约束 cascade/restrict;

alter table 表名 alter column 列名 数据类型;
```

#### 删除基本表

```sql
drop table 表名 cascade/restrict;
```

### 索引

```sql
#唯一索引
create unique index 索引名 on 表名(列名1 次序, 列名n 次序);

#聚簇索引
create cluster index 索引名 on 表名(列名1 次序, 列名n 次序);

ASC -- 升序
DESC -- 降序

#修改索引
alter index 旧索引名 rename to 新索引名;

#删除索引
drop index 索引名;
```

## SELECT语句

```sql
select distinct/all 属性
from 表/视图
where 条件表达式
group by 列名 having 条件表达式
order by 列名 次序/默认ASC;
```

### 基本查询

``` sql
select
* -- 全部列
表达式
列名 别名
distinct 列名 -- 去掉重复列

#聚集函数
-- 列数
count(*)
count(distinct/all 列名)
-- 值的平均数
avg(distinct/all 列名)
-- 总和
sum(distinct/all 列名)
-- 最大值/最小值
max/min(distinct/all 列名)

where 列名
-- 比较大小
= > < >= <= !=(<>)
-- 范围
between a and b
-- 确定集合
in ('分量1', '分量n')
-- 字符匹配
like '字符串' escape '\'
not like ''
#通配符
# % 0~任意长度
# _ 单个字符, 一个汉字(两个字符)用两个_
# escape定义转义字符
-- 空值查询
is null
is not null
-- and or 多条件

group by 列名1
group by 列名 having 条件

order by 列名1, 列名2
```

where与having区别

where用来从基本表与视图中选择条件，不接聚集函数

having用于从组中选择条件

### 连接查询

```sql
#两表连接查询
where 表名1.列名1 比较运算符 表名2.列名2;
```


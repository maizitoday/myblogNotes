---
title:       "mysql系列-如何分析sql语句执行慢"
subtitle:    ""
description: "慢日志,explain执行任务，profiling查询分析"
date:        2019-08-26
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["mysql系列", "慢日志,explain执行任务，profiling查询分析"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：<https://blog.csdn.net/jack__frost/article/details/71512404>**

# 明确搜索优化的整体思路 

索引优化，查询优化，查询缓存，服务器设置优化，操作系统和硬件优化，应用层面优化（web服务器，缓存）等等。对于一个整体项目而言只有这些齐头并进，才能实现mysql高性能。

# 查询优化的因素

## 是否向数据库请求了不需要的数据

也就是说不要轻易使用select * from ，能明确多少数据就查多少个。

## mysql是否扫描额外的纪录

查询是否扫描了过多的数据。最简单的衡量查询开销三个指标如下：响应时间；扫描的行数；返回的行数。

没有哪个指标能够完美地衡量查询的开销，但它们大致反映了mysql在内部执行查询时需要多少数据，并可以推算出查询运行的时间。

**这三个指标都会记录到mysql的慢日志中，所以检查慢日志记录是找出扫描行数过多的查询的好办法。**

### 响应时间

是两个部分之和：服务时间和排队时间。服务时间是指数据库处理这个查询真正花了多长时间。 排队时间是指服务器因为等待某些资源而没有真正执行查询的时间。—可能是等io操作完成，也可能是等待行锁，等等。

### 扫描的行数和返回的行数

：分析查询时，查看该查询扫描的行数是非常有帮助的。这在一定程度上能够说明该查询找到需要的数据的效率高不高。

### 扫描的行数和访问类型

在**expain**语句中的type列反应了访问类型。访问类型有很多种，从全表扫描（ALL）到索引扫描（index）到范围扫描（）到唯一索引查询到常数引用等。这里列的这些，速度由慢到快，扫描的行数也是从小到大。

**如果发现查询需要扫描大量的数据但只返回少数的行，那么通常可以尝试下面的技巧去优化它：**

1. 使用索引覆盖扫描。
2. 改变库表结构。例如使用单独的汇总表。
3. 重写这个复杂的查询。让mysql优化器能够以更优化的方式执行这个查询。



## 查询的流程

![cb39a58ccf17cc1b96db94010b9cd47e955c83e4](/img/cb39a58ccf17cc1b96db94010b9cd47e955c83e4.png)

# 优化查询前的几个工具

## 查看MySQL整体状态

```sql
Mysql> show status; ——显示状态信息（扩展show status like ‘XXX’）
Mysql>show variables ——显示系统变量（扩展show variables like ‘XXX’）
Mysql>show innodb status ——显示InnoDB存储引擎的状态
Mysql>show processlist ——查看当前SQL执行，包括执行状态、是否锁表等
```

```shell
Shell> mysqladmin variables -u username -p password——显示系统变量
Shell> mysqladmin extended-status -u username -p password——显示状态信息
Shell> mysqld –verbose –help [|more #逐行显示] 查看状态变量及帮助：
```



## 慢查询

```shell
# 查看慢查日志是否开启
SHOW VARIABLES  LIKE 'slow_query_log';
SET GLOBAL slow_query_log=on;

# 查看是否没有使用索引的查询记录
SHOW VARIABLES LIKE 'log%';
# 开启所有查询都记录下来
SET GLOBAL log_queries_not_using_indexes=on;

#查看执行多长时间的语句记录下来。
SHOW VARIABLES LIKE 'long_query_time';
#这个地方设置后，需要重新开启一个终端才可以看到效果
set global long_query_time=0;

#查看慢日志的具体文件
SHOW  VARIABLES LIKE 'slow%'
```

### 查询日志格式

```shell
#查看慢日志的具体文件
SHOW  VARIABLES LIKE 'slow%'

SET timestamp=1558163795;
/* ApplicationName=DataGrip 2017.2.2 */ SET SQL_SELECT_LIMIT=12;
# User@Host: root[root] @  [172.17.0.1]  Id:    78
# Query_time: 0.000247  Lock_time: 0.000102 Rows_sent: 1  Rows_examined: 2
SET timestamp=1558163795;
/* ApplicationName=DataGrip 2017.2.2 */ SELECT * FROM work.PDMAN_DB_VERSION WHERE DB_VERSION = 'v1.0.4';
```



### 开启慢查询日志

#### 配置文件修改

配置文件修改，在配置文件my.cnf或my.ini中在[mysqld]一行下面加入两个配置参数。

```shell
log-slow-queries={自己想存放的日志路径}/slow-query.log
long_query_time=10 //表示查询超过10秒才记录；
```

注：log-slow-queries参数为慢查询日志存放的位置，一般这个目录要有mysql的运行帐号的可写权限，一般都将这个目录设置为mysql的数据存放目录；



#### 查看日志启动状态

```sql
show variables like “slow%”;
```

| slow_launch_time    | 2                                    |
| ------------------- | ------------------------------------ |
| slow_query_log      | OFF                                  |
| slow_query_log_file | /var/lib/mysql/63c0513730d7-slow.log |



#### 设置慢日志开启

```sql
set global slow_query_log = ON;
```

| slow_launch_time    | 2                                    |
| ------------------- | ------------------------------------ |
| slow_query_log      | ON                                   |
| slow_query_log_file | /var/lib/mysql/63c0513730d7-slow.log |



#### 查询long_query_time 的值

```sql
show variables like “long%”;
```

| long_query_time | 10.000000 |
| --------------- | --------- |
|                 |           |



### explain查询分析

![5-19-3](/img/5-19-3.png)

![5-19-4](/img/5-19-4.png)

使用 EXPLAIN 关键字可以模拟优化器执行SQL查询语句，通过explain命令可以得到:

1. 表的读取顺序
2. 数据读取操作的操作类型
3. 哪些索引可以使用
4. 哪些索引被实际使用
5. 表之间的引用
6. 每张表有多少行被优化器查询

示例如下：

```sql
ALTER TABLE school.student add index age_index (age); // 添加索引 
explain select * from school.student where age = 5;
```

| select_type | Table   | partitions | Type | possible_keys | Key       | key_len | ref   | rows | filtered | Extra |
| ----------- | ------- | ---------- | ---- | ------------- | --------- | ------- | ----- | ---- | -------- | ----- |
| SIMPLE      | student |            | ref  | age_index     | age_index | 5       | const | 1    | 100.0    |       |



#### select_type

查询中每个select子句的类型，

1. SIMPLE(简单SELECT,不使用UNION或子查询等)
2. PRIMARY(查询中若包含任何复杂的子部分,最外层的select被标记为PRIMARY)
3. UNION(UNION中的第二个或后面的SELECT语句)
4. DEPENDENT UNION(UNION中的第二个或后面的SELECT语句，取决于外面的查询)
5. UNION RESULT(UNION的结果)
6. SUBQUERY(子查询中的第一个SELECT)
7. DEPENDENT SUBQUERY(子查询中的第一个SELECT，取决于外面的查询)
8. DERIVED(派生表的SELECT, FROM子句的子查询)
9. UNCACHEABLE SUBQUERY(一个子查询的结果不能被缓存，必须重新评估外链接的第一行)
  

#### Table

显示这一行的数据是关于哪张表的。

#### type：这是最重要的字段之一

显示查询使用了何种类型。从最好到最差的连接类型为NULL、system、const、eq_ref、ref、range、index和ALL。

##### NULL

MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。

##### system、const

可以将查询的变量转为常量. 如id=1; id为 主键或唯一键。当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量,system是const类型的特例，当查询的表只有一行的情况下，使用system。

##### eq_ref

访问索引,返回某单一行的数据.(通常在联接时出现，查询使用的索引为主键或惟一键)。类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件。

##### ref

访问索引,返回某个值的数据.(可以返回多行) 通常使用=时发生。表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值。

##### range

这个连接类型使用索引返回一个范围中的行，比如使用>或<查找东西，并且该字段上建有索引时发生的情况(注:不一定好于index)。只检索给定范围的行，使用一个索引来选择行。

##### index

以索引的顺序进行全表扫描，优点是不用排序,缺点是还要全表扫描。index与ALL区别为index类型只遍历索引树。

##### ALL

全表扫描，应该尽量避免。 MySQL将遍历全表以找到匹配的行。



#### possible_keys

显示可能应用在这张表中的索引。如果为空，没有可能的索引。可以为相关的域从WHERE语句中选择一个合适的语句。如果该列是NULL，则没有相关的索引。在这种情况下，可以通过检查WHERE子句看是否它引用某些列或适合索引的列来提高你的查询性能。如果是这样，创造一个适当的索引并且再次用EXPLAIN检查查询。

#### key

实际使用的索引。如果为NULL，则没有使用索引。

#### key_len

在不损失精确性的情况下，长度越短越好。表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度（key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的）。

#### ref

显示索引的哪一列被使用了，如果可能的话，是一个常数。

#### rows

MySQL认为必须检索的用来返回请求数据的行数，表示MySQL根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行数。



#### Extra：关于MYSQL如何解析查询的额外信息

##### using index

只用到索引,可以避免访问表.。表示查询在索引树中就可查找所需数据, 不用扫描表数据文件, **往往说明性能不错**。

##### using where

使用到where来过虑数据. 不是所有的where clause都要显示using where. 如以=方式访问索引.

##### using tmporary

查询有使用临时表, 一般出现于排序, 分组和多表 join 的情况, 查询效率不高, **建议优化**.

##### using filesort

用到额外的排序. (当使用order by v1,而没用到索引时,就会使用额外的排序)。MySQL中无法利用索引完成的排序操作称为“文件排序”

##### range checked for eache record(index map:N)

没有好的索引.

##### Using join buffer

改值强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。**如果出现了这个值，那应该注意，根据查询的具体情况可能需要添加索引来改进能。**

##### Impossible where

这个值强调了where语句会导致没有符合条件的行。



## profiling查询分析

通过慢日志查询可以知道哪些SQL语句执行效率低下，通过explain我们可以得知SQL语句的具体执行情况，索引使用等，还可以结合show命令查看执行状态。

如果觉得explain的信息不够详细，可以同通过profiling命令得到更准确的SQL执行消耗系统资源的信息。

### 打开profiling查询分析

profiling默认是关闭的。可以通过以下语句进行处理查看：

```sql
select @@profiling;  // 查看 
 
set profiling = 1;  //  打开 

select * from school.student where age = 5; // 写几条sql

```

进行查看具体sql信息如下：

```sql
show profiles; //  然后查看最近SQL，可以得到被执行的SQL语句的时间和ID
```

| 12   | 0.00438175 | select * from school.student where age = 5 |
| ---- | ---------- | ------------------------------------------ |
| 13   | 3.035E-4   | SHOW WARNINGS                              |
| 14   | 5.885E-4   | SHOW WARNINGS                              |
| 15   | 2.7525E-4  | show profiles\G                            |
| 16   | 3.92E-4    | SHOW WARNINGS                              |
| 17   | 1.9925E-4  | show profiles\G                            |
| 18   | 3.375E-4   | SHOW WARNINGS                              |
| 19   | 2.365E-4   | SHOW WARNINGS                              |
| 20   | 2.83E-4    | SHOW WARNINGS                              |
| 21   | 3.31E-4    | SHOW WARNINGS                              |
| 22   | 3.36E-4    | SHOW WARNINGS                              |
| 23   | 3.1E-4     | SHOW WARNINGS                              |
| 24   | 0.00100725 | SHOW WARNINGS                              |
| 25   | 2.9525E-4  | SHOW WARNINGS                              |
| 26   | 3.77E-4    | SHOW WARNINGS                              |

```sql
show profile for query 12;  // 得到对应SQL语句执行的详细信息
```

| starting                      | 0.000084 |
| ----------------------------- | -------- |
| Executing hook on transaction | 0.000011 |
| starting                      | 0.000016 |
| checking permissions          | 0.000013 |
| Opening tables                | 0.000072 |
| init                          | 0.000023 |
| System lock                   | 0.000037 |
| optimizing                    | 0.000027 |
| statistics                    | 0.000127 |
| preparing                     | 0.000037 |
| executing                     | 0.000091 |
| end                           | 0.000018 |
| query end                     | 0.000014 |
| waiting for handler commit    | 0.000027 |
| closing tables                | 0.000062 |
| freeing items                 | 0.000283 |
| cleaning up                   | 0.000049 |

---
title:       "MySQL数据库"
subtitle:    ""
description: ""
date:        2019-05-19
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-05-23-service_2_service_auth/background.jpg"
tags:        ["数据库", "性能优化"]
categories:  ["Tech" ]
---

[TOC]



**说明： 文章里面的图片来源地址:https://www.imooc.com/learn/194,教学视频**

# schema是什么

**原文：https://blog.csdn.net/hpulfc/article/details/79564790** 

在SQL环境下，schema就是数据库对象的集合，所谓的数据库对象也就是常说的表，索引，视图，存储过程等。在schema之上的，就是数据库的实例，也就是通常create databases获得的东西。也就是说一个schema 实例 可以有多个schema, 可以给不同的用户创建不同的schema,并且他们都是在同一数据库实例下面。

在MySQL中基本认为schema和数据库相同，也就是说schema的名称和数据库的实例的名称相同，一个数据库有一个schema。

而在PostgreSQL中，可以创建一个数据库，然后在数据库中，创建不同的schema,每个schema又有着一些各自的表，索引等。

# 数据库事物

**转载地址： https://www.cnblogs.com/huanongying/p/7021555.html**

## 事务的基本要素（ACID）

### 原子性（Atomicity）

事务开始后所有操作，要么全部做完，要么全部不做，不可能停滞在中间环节。事务执行过程中出错，会回滚到事务开始前的状态，所有的操作就像没有发生一样。也就是说事务是一个不可分割的整体，就像化学中学过的原子，是物质构成的基本单位。

### 一致性（Consistency）

事务开始前和结束后，数据库的完整性约束没有被破坏 。比如A向B转账，不可能A扣了钱，B却没收到。

### 隔离性（Isolation）

同一时间，只允许一个事务请求同一数据，不同的事务之间彼此没有任何干扰。比如A正在从一张银行卡中取钱，在A取钱的过程结束前，B不能向这张卡转账。

### 持久性（Durability）

事务完成后，事务对数据库的所有更新将被保存到数据库，不能回滚。

## 事务的并发问题

### 脏读

事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据。

### 不可重复读

事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果 不一致。

### 幻读

系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读。



## MySQL事务隔离级别

转载地址：https://blog.csdn.net/xuheng8600/article/details/79869669

| 事务隔离级别                 | 脏读 | 不可重复读 | 幻读 |
| ---------------------------- | ---- | ---------- | ---- |
| 读未提交（read-uncommitted） | 是   | 是         | 是   |
| 不可重复读（read-committed） | 否   | 是         | 是   |
| 可重复读（repeatable-read）  | 否   | 否         | 是   |
| 串行化（serializable）       | 否   | 否         | 否   |



### 事物级别查看和修改

```sql
 #查看当前会话隔离级别
 SELECT @@tx_isolation
 
 #设置当前会话隔离级别
 set session transaction isolation level repeatable read;
 
 #查看系统当前隔离级别
 select @@global.tx_isolation;
 
 #查看系统当前隔离级别
 set global transaction isolation level repeatable read;
```



### 隔离级别的理解

#### 读未提交（read-uncommitted）

可以看到未提交的数据（脏读），举个例子：别人说的话你都相信了，但是可能他只是说说，并不实际做。

#### 不可重复读（read-committed）

读取提交的数据。但是，可能多次读取的数据结果不一致（不可重复读，幻读）。用读写的观点就是：读取的行数据，可以写。

#### repeatable read(MySQL默认隔离级别)

可以重复读取，但有幻读。读写观点：读取的数据行不可写，但是可以往表中新增数据。在MySQL中，其他事务新增的数据，看不到，不会产生幻读。采用多版本并发控制（MVCC）机制解决幻读问题。

#### serializable

可读，不可写。像java中的锁，写数据必须等待另一个事务结束。



# 数据库锁

**转载地址：https://www.jianshu.com/p/2633fc36b57a**

## 乐观锁

乐观锁（Optimistic Lock），顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在提交更新的时候会判断一下在此期间别人有没有去更新这个数据。乐观锁适用于读多写少的应用场景，这样可以提高吞吐量。

乐观锁：假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性。

乐观锁一般来说有以下2种方式：

- 使用数据版本（Version）记录机制实现，这是乐观锁最常用的一种实现方式。何谓数据版本？即为数据增加一个版本标识，一般是通过为数据库表增加一个数字类型的 “version” 字段来实现。当读取数据时，将version字段的值一同读出，数据每更新一次，对此version值加一。当我们提交更新的时候，判断数据库表对应记录的当前版本信息与第一次取出来的version值进行比对，如果数据库表当前版本号与第一次取出来的version值相等，则予以更新，否则认为是过期数据。
- 使用时间戳（timestamp）。乐观锁定的第二种实现方式和第一种差不多，同样是在需要乐观锁控制的table中增加一个字段，名称无所谓，字段类型使用时间戳（timestamp）, 和上面的version类似，也是在更新提交的时候检查当前数据库中数据的时间戳和自己更新前取到的时间戳进行对比，如果一致则OK，否则就是版本冲突。
   Java JUC中的atomic包就是乐观锁的一种实现，AtomicInteger 通过CAS（Compare And Set）操作实现线程安全的自增。

### 优点与不足

乐观并发控制相信事务之间的数据竞争(data race)的概率是比较小的，因此尽可能直接做下去，直到提交的时候才去锁定，所以**不会产生任何锁和死锁**。但如果直接简单这么做，还是有可能会遇到不可预期的结果，例如两个事务都读取了数据库的某一行，经过修改以后写回数据库，这时就遇到了问题。

 

## 悲观锁

悲观锁（Pessimistic Lock），顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。

悲观锁：假定会发生并发冲突，屏蔽一切可能违反数据完整性的操作。

Java synchronized 就属于悲观锁的一种实现，每次线程要修改数据时都先获得锁，保证同一时刻只有一个线程能操作数据，其他线程则会被block。

### 优点与不足

 悲观并发控制实际上是“先取锁再访问”的保守策略，**为数据处理的安全提供了保证**。但是在效率方面，处理加锁的机制**会让数据库产生额外的开销，还有增加产生死锁的机会**；另外，在只读型事务处理中由于不会产生冲突，也没必要使用锁，这样做只能增加系统负载；还有会**降低了并行性**，一个事务如果锁定了某行数据，其他事务就必须等待该事务处理完才可以处理那行数。

# 表空间

**转载地址：https://blog.csdn.net/daohengshangqian/article/details/50116493**

mysql 表空间管理与共享维护，INNODB 对于表的存储有两种形式，一种是**共享表空间**，及多张表放在一个文件中，还有一种是**独立表空间**，每个表都有独立的数据文件。



## 共享表空间



### 查看当前共享表空间

```sql
show variables like '%innodb_data_file_path%' ;
```



### 增加共享表空间

```shell
[root@localhost mysql]# cat /etc/my.cnf 
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
innodb_data_file_path=ibdata1:12M;ibdata2:24M:autoextend
```

更改后，可以再来查来执行 **show variables like '%innodb_data_file_path%' ;**



## 独立表空间

```yaml
show variables like '%innodb_file_per_table%' ;
```

# 索引 

**原文地址： https://www.cnblogs.com/bypp/p/7755307.html**

## 什么是索引？

一般的应用系统，读写比例在10:1左右，而且插入操作和一般的更新操作很少出现性能问题，在生产环境中，我们遇到最多的，也是最容易出问题的，还是一些复杂的查询操作，因此对查询语句的优化显然是重中之重。说起加速查询，就不得不提到索引了。

## 为什么要有索引呢？

索引在MySQL中也叫做“键”，是存储引擎用于快速找到记录的一种数据结构。索引对于良好的性能，非常关键，尤其是当表中的数据量越来越大时，索引对于性能的影响愈发重要。索引优化应该是对查询性能优化最有效的手段了。索引能够轻易将查询性能提高好几个数量级。索引相当于字典的音序表，如果要查某个字，如果不使用音序表，则需要从几百页中逐页去查。

## 索引类型

### 常规索引

常规索引，也叫普通索引（index或key），它可以常规地提高查询效率。一张数据表中可以有多个常规索引。常规索引是使用最普遍的索引类型，如果没有明确指明索引的类型，我们所说的索引都是指常规索引。



### 主键索引

主键索引（Primary Key），也简称主键。它可以提高查询效率，并提供唯一性约束。一张表中只能有一个主键。被标志为自动增长的字段一定是主键，但主键不一定是自动增长。一般把主键定义在无意义的字段上（如：编号），主键的数据类型最好是数值。

### 唯一索引

唯一索引（Unique Key），可以提高查询效率，并提供唯一性约束。一张表中可以有多个唯一索引。

### 全文索引

全文索引（Full Text），可以提高全文搜索的查询效率，一般使用Sphinx替代。但Sphinx不支持中文检索，Coreseek是支持中文的全文检索引擎，也称作具有中文分词功能的Sphinx。实际项目中，我们用到的是Coreseek。

### 外键索引

外键索引（Foreign Key），简称外键，它可以提高查询效率，外键会自动和对应的其他表的主键关联。外键的主要作用是保证记录的一致性和完整性。



## 创建/删除索引

###  语法

```sql
#方法一：创建表时
    　　CREATE TABLE 表名 (
                字段名1  数据类型 [完整性约束条件…],
                字段名2  数据类型 [完整性约束条件…],
                [UNIQUE | FULLTEXT | SPATIAL ]   INDEX | KEY
                [索引名]  (字段名[(长度)]  [ASC |DESC]) 
                );


#方法二：CREATE在已存在的表上创建索引
        CREATE  [UNIQUE | FULLTEXT | SPATIAL ]  INDEX  索引名 
                     ON 表名 (字段名[(长度)]  [ASC |DESC]) ;


#方法三：ALTER TABLE在已存在的表上创建索引
        ALTER TABLE 表名 ADD  [UNIQUE | FULLTEXT | SPATIAL ] INDEX
                             索引名 (字段名[(长度)]  [ASC |DESC]) ;
                             
#删除索引：DROP INDEX 索引名 ON 表名字;
```

### demo实例

```sql
创建索引
    -在创建表时就创建（需要注意的几点）
    create table s1(
    id int ,#可以在这加primary key
    #id int index #不可以这样加索引，因为index只是索引，没有约束一说，
    #不能像主键，还有唯一约束一样，在定义字段的时候加索引
    name char(20),
    age int,
    email varchar(30)
    #primary key(id) #也可以在这加
    index(id) #可以这样加
    );
    -在创建表后在创建
    create index name on s1(name); #添加普通索引
    create unique age on s1(age);添加唯一索引
    alter table s1 add primary key(id); #添加住建索引，也就是给id字段增加一个主键约束
    create index name on s1(id,name); #添加普通联合索引
删除索引
    drop index id on s1;
    drop index name on s1; #删除普通索引
    drop index age on s1; #删除唯一索引，就和普通索引一样，不用在index前加unique来删，直接就可以删了
    alter table s1 drop primary key; #删除主键(因为它添加的时候是按照alter来增加的，那么我们也用alter来删)
```



### 索引的优化

![5-19-1](/img/5-19-1.png)

![5-19-2](/img/5-19-2.png)



# mysql三种虚拟表

- 临时表

- 内存表

- 视图

  

## 临时表

**原文：https://blog.csdn.net/qq_41376740/article/details/79393943**

### 什么是临时表

MySQL用于存储一些中间结果集的表，临时表只在当前连接可见，当关闭连接时，Mysql会自动删除表并释放所有空间。为什么会产生临时表：一般是由于复杂的SQL导致临时表被大量创建。临时表是建立在系统临时文件夹中的表。临时表的数据和表结构都存储在内存之中，**退出的时候所占的空间会被释放**

### 临时表的应用场景

当工作在十分大的表上运行时，运行相关查询，来获的一个大量数据的小的子集。较好的办法，不是对整个表运行这些查询，而是让MySQL每次找出所需的少数记录，将记录选择到一个临时表，然后对这些表运行查询。

### 创建临时表

```sql
关键字为temporary

create temporary table tmp_table(
       name varchar(10) not null,
       value int not null
);
注意:
show tables;
show table status
这两个命令无法查看临时表。 但是可以查内存表
```



### 直接将查询结果导入临时表

```sql
CREATE TEMPORARY TABLE tmp_table SELECT * FROM table_name
```



## 视图

**原文：https://www.cnblogs.com/sustudy/p/4166714.html**

### 作用



#### 提高了重用性

 提高了重用性，就像一个函数。如果要频繁获取user的name和goods的name。就应该使用以下sql语言。示例：

```sql
select a.name as username, b.name as goodsname from user as a, goods as b, ug as c where a.id=c.userid and c.goodsid=b.id;

但有了视图就不一样了，创建视图other。示例
        create view other as select a.name as username, b.name as goodsname from user as a, goods as b, ug as c where a.id=c.userid and c.goodsid=b.id;

创建好视图后，就可以这样获取user的name和goods的name。示例：
        select * from other;
以上sql语句，就能获取user的name和goods的name了。
```



#### 对数据库重构，却不影响程序的运行

 对数据库重构，却不影响程序的运行。假如因为某种需求，需要将user拆房表usera和表userb，该两张表的结构如下：

```sql
测试表:usera有id，name，age字段
测试表:userb有id，name，sex字段
这时如果php端使用sql语句：select * from user;那就会提示该表不存在，这时该如何解决呢。
解决方案：创建视图。以下sql语句创建视图：
        create view user as select a.name,a.age,b.sex from usera as a, userb as b where a.name=b.name;
以上假设name都是唯一的。此时php端使用sql语句：select * from user;就不会报错什么的。这就实现了更改数据库结构，不更改脚本程序的功能了。
```



#### 提高了安全性能。可以对不同的用户

提高了安全性能。可以对不同的用户，设定不同的视图。例如：某用户只能获取user表的name和age数据，不能获取sex数据。则可以这样创建视图。示例如下：

```sql
create view other as select a.name, a.age from user as a;
这样的话，使用sql语句：select * from other; 最多就只能获取name和age的数据，其他的数据就获取不了了。
```



#### 让数据更加清晰

让数据更加清晰。想要什么样的数据，就创建什么样的视图。经过以上三条作用的解析，这条作用应该很容易理解了吧

# 开启慢查询日志

可以记录超时的所有的SQL语句

```sql
#开启终端，连接客户端
mysql -u root -p

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

## 查询日志格式

```sql
#查看慢日志的具体文件
SHOW  VARIABLES LIKE 'slow%'

SET timestamp=1558163795;
/* ApplicationName=DataGrip 2017.2.2 */ SET SQL_SELECT_LIMIT=12;
# User@Host: root[root] @  [172.17.0.1]  Id:    78
# Query_time: 0.000247  Lock_time: 0.000102 Rows_sent: 1  Rows_examined: 2
SET timestamp=1558163795;
/* ApplicationName=DataGrip 2017.2.2 */ SELECT * FROM work.PDMAN_DB_VERSION WHERE DB_VERSION = 'v1.0.4';
```

# EXPLAIN分析

![5-19-3](/img/5-19-3.png)

![5-19-4](/img/5-19-4.png)

![5-19-5](/img/5-19-5.png)

 

# 分库分表

![5-19-6](/img/5-19-6.jpg)

![5-19-7](/img/5-19-7.jpg) 



# 数据库设计

## 范式化和反范式化

![5-19-8](/img/5-19-8.png)

![5-19-9](/img/5-19-9.png)

![5-19-10](/img/5-19-10.png)



# 数据库系统配置

![5-19-11](/img/5-19-11.jpg)



# sql left right inner 使用

**原文：https://blog.csdn.net/weixin_41926488/article/details/80327541**



## 左链接 left join on

left join,在两张表进行连接查询时，会返回左表所有的行，即使在右表中没有匹配的记录。

```sql
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
LEFT JOIN Orders
ON Persons.Id_P=Orders.Id_P
ORDER BY Persons.LastName
```



## 右链接 right join on

right join,在两张表进行连接查询时，会返回右表所有的行，即使在左表中没有匹配的记录。

```sql
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
RIGHT JOIN Orders
ON Persons.Id_P=Orders.Id_P
ORDER BY Persons.LastName
```



## 内链接  inner join

inner join，在两张表进行连接查询时，只保留两张表中完全匹配的结果集。

```sql
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
INNER JOIN Orders
ON Persons.Id_P=Orders.Id_P
ORDER BY Persons.LastName
```



## 全链接 full join

full join,在两张表进行连接查询时，返回左表和右表中所有没有匹配的行。

```sql
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
FULL JOIN Orders
ON Persons.Id_P=Orders.Id_P
ORDER BY Persons.LastName
```

 


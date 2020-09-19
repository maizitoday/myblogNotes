---
title:       "mysql系列-数据库存储引擎"
subtitle:    ""
description: "InnoDB, MyISAM, Mrg_Myisam，3种存储引擎对比"
date:        2019-08-28
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["mysql系列", "InnoDB, MyISAM, Mrg_Myisam，3种存储引擎对比"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：<https://blog.csdn.net/jack__frost/article/details/72904318**>

# 为什么要合理选择数据库存储引擎

MySQL中的数据用各种不同的技术存储在文件（或者内存）中。这些技术中的每一种技术都使用不同的存储机制、索引技巧、锁定水平并且最终提供广泛的不同的功能和能力。通过选择不同的技术，你能够获得额外的速度或者功能，从而改善你的应用的整体功能。

这些不同的技术以及配套的相关功能在MySQL中被称作存储引擎(也称作表类型)。MySQL默认配置了许多不同的存储引擎，可以预先设置或者在MySQL服务器中启用。你可以选择适用于服务器、数据库和表格的存储引擎，以便在选择如何存储你的信息、如何检索这些信息以及你需要你的数据结合什么性能和功能的时候为你提供最大的灵活性。

## 定义

数据库引擎是用于存储、处理和保护数据的核心服务。利用数据库引擎可控制访问权限并快速处理事务，从而满足企业内大多数需要处理大量数据的应用程序的要求。 使用数据库引擎创建用于联机事务处理或联机分析处理数据的关系数据库。这包括创建用于存储数据的表和用于查看、管理和保护数据安全的数据库对象（如索引、视图和存储过程）。


## 如何修改数据库引擎

### 1.  修改配置文件my.ini

将mysql.ini另存为my.ini，在[mysqld]后面添加default-storage-engine=InnoDB，重启服务，数据库默认的引擎修改为InnoDB

### 2.   在建表的时候指定

```sql
create table testMyIsam(  
id int unsigned primary key auto_increment,  
name varchar(20) not null  
)engine=myisam;  
```

### 3.   建表后更改

```sql
alter table testMyIsam engine = InnoDB;
```

### 4.  查看修改成功

```sql
show create table table_name
```

### 5.  查看当前支持的引擎

可以看到常用的 MyISAM 和 默认的 InnoDB ，一个不支持事物， 一个支持。 

```sql
show engines;
```

| Engine             | Support | Comment                                                      | Transactions | XA   | Savepoints |
| ------------------ | ------- | ------------------------------------------------------------ | ------------ | ---- | ---------- |
| FEDERATED          | NO      | Federated MySQL storage engine                               |              |      |            |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables    | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys   | YES          | YES  | YES        |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                           | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                        | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                        | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                           | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                       | NO           | NO   | NO         |

 

# 常用三种引擎

## InnoDB引擎（默认的存储引擎）

### 定义

Innodb引擎提供了对数据库ACID事务的支持，并且实现了SQL标准的四种隔离级别，该引擎还提供了行级锁和外键约束，它的设计目标是处理大容量数据库系统，它本身其实就是基于MySQL后台的完整数据库系统，MySQL运行时Innodb会在内存中建立缓冲池，用于缓冲数据和索引。但是该引擎不支持FULLTEXT类型的索引，而且它没有保存表的行数，当SELECT COUNT(*) FROM TABLE时需要扫描全表。**当需要使用数据库事务时，该引擎当然是首选。**由于锁的粒度更小，写操作不会锁定全表，所以在并发较高时，使用Innodb引擎会提升效率。但是使用行级锁也不是绝对的，如果在执行一个SQL语句时MySQL不能确定要扫描的范围，InnoDB表同样会锁全表。

```sql
//这个就是select锁表的一种，不明确主键。增删改查都可能会导致锁全表，在以后我们会详细列出。
SELECT * FROM products WHERE name='Mouse' FOR UPDATE;
```

### 适用场景

#### 1.  经常更新的表，适合处理多重并发的更新请求。

#### 2.  支持事务。

#### 3.  可以从灾难中恢复（通过bin-log日志等）。

#### 4.  外键约束。只有他支持外键。

#### 5.  支持自动增加列属性auto_increment。

**innodb`引擎的`insert,update,delete`操作都会给操作数据加上排他锁(行级锁).这时候其他事务是没法对这行数据进行操作的.**



## MyIsam

### 定义

它没有提供对数据库事务的支持，也不支持行级锁和外键，因此当INSERT(插入)或UPDATE(更新)数据时即写操作需要锁定整个表，效率便会低一些。引擎在创建表的时候，会创建三个文件，一个是.frm文件用于存储表的定义，一个是.MYD文件用于存储表的数据，另一个是.MYI文件，存储的是索引。操作系统对大文件的操作是比较慢的，这样将表分为三个文件，那么.MYD这个文件单独来存放数据自然可以优化数据库的查询等操作。有索引管理和字段管理。MyISAM还使用一种表格锁定的机制，来优化多个并发的读写操作，其代价是你需要经常运行OPTIMIZE TABLE命令，来恢复被更新机制所浪费的空间。

### 适用场景

#### 1.  不支持事务的设计，可以在service层进行根据自己的业务需求进行相应的控制。

#### 2.  不支持外键的表设计。

#### 3.  查询速度很快，如果数据库insert和update的操作比较多的话比较适用。

#### 4.   整天 对表进行加锁的场景。

#### 5.   MyISAM极度强调快速读取操作。

#### 6.   MyIASM中存储了表的行数，于是SELECT COUNT(*) FROM TABLE时只需要直接读取已经保存好的值而不需要进行全表扫描。如果表的读操作远远多于写操作且不需要数据库事务的支持，那么MyIASM也是很好的选择。

#### 7.   ISAM执行读取操作的速度很快，而且不占用大量的内存和存储资源。



### 缺点

#### 1.  就是不能在表损坏后恢复数据。（是不能主动恢复）

#### 2.  也不能够容错。

如果你的硬盘崩溃了，那么数据文件就无法恢复了。如果你正在把ISAM用在关键任务应用程序里，那就必须经常备份你所有的实 时数据，通过其复制特性，MYSQL能够支持这样的备份应用程序。



## Mrg_Myisam(分表的一种方式–水平分表)

### 定义

是一个相同的可以被当作一个来用的MyISAM表的集合。“相同”意味着所有表同样的列和索引信息。

也就是说，他将MyIsam引擎的多个表聚合起来，但是他的内部没有数据，真正的数据依然是MyIsam引擎的表中，但是可以直接进行查询、删除更新等操作。

比如：我们可能会遇到这样的问题，同一种类的数据会根据数据的时间分为多个表，如果这时候进行查询的话，就会比较麻烦，Merge可以直接将多个表聚合成一个表统一查询，然后再删除Merge表（删除的是定义），原来的数据不会影响。

**注意：我们现在可以用MyCat数据库中间件来进行分库分表处理。** 

# InnoDB和MyIsam使用及其原理对比

## 1.  百万级插入

对比插入效率（百万级插入）：（虽然速度上MyISAM快，但是增删改是涉及事务安全的，所以用InnoDB相对好很多）

## 2.  对比更新

虽然速度上MyISAM快，但是增删改是涉及事务安全的，所以InnoDB相对好很多。

## 3.  查询总数目

```sql
// 都是100万数据，查询时间如下， 我这个表字段很少只有2个， 字段多差距更大。
select count(*) from testInnoDB;   // 0.1 秒
select count(*) from school.testMyIsam; // 0.002 秒
```

字段多， 差距会更大。 



## 4.  存储大小

testMyIsam 存了一百万。testinnodb 存了两万。

![20181209204205186](/img/20181209204205186.png)

## 效果对比总述

### 1.  事务

MyISAM类型不支持事务处理等高级处理，而InnoDB类型支持，提供事务支持已经外部键等高级数据库功能。

### 2.  性能主题

MyISAM类型的表强调的是性能，其执行数度比InnoDB类型更快。

### 3.  行数保存

InnoDB 中不保存表的具体行数，也就是说，执行select count(*) fromtable时，InnoDB要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。注意的是，当count(*)语句包含where条件时，两种表的操作是一样的。

### 4.  索引存储

对于AUTO_INCREMENT类型的字段，InnoDB中必须包含只有该字段的索引，但是在MyISAM表中，可以和其他字段一起建立联合索引。

**MyISAM支持全文索引（FULLTEXT）、压缩索引，InnoDB不支持**

### 5.  服务器数据备份

InnoDB必须导出SQL来备份，InnoDB是拷贝数据文件、备份 binlog，或者用 mysqldump，在数据量达到几十G的时候就相对痛苦了。

MyISAM的数据是以文件的形式存储，所以在跨平台的数据转移中会很方便。在备份和恢复时可单独针对某个表进行操作。

### 6.  锁的支持

MyISAM只支持表锁。InnoDB支持表锁、行锁 行锁大幅度提高了多用户并发操作的新能。但是InnoDB的行锁，只是在WHERE的主键是有效的，非主键的WHERE都会锁全表的



## 使用建议

### 以下两点**必须使用** InnoDB

1. 可靠性高或者要求事务处理，则使用InnoDB。这个是必须的。
2.  表更新和查询都相当的频繁，并且**表锁定的机会比较大**的情况指定InnoDB数据引擎的创建

### MyISAM的使用场景

1. 做很多count的计算的。如一些日志，调查的业务表。
2. 插入修改不频繁，查询非常频繁的。



# InnoDB和MyIsam引擎原理

## MyIASM引擎的索引结构

MyISAM索引结构: MyISAM索引用的B+ tree来储存数据，MyISAM索引的指针指向的是键值的地址，地址存储的是数据。

B+Tree的数据域存储的内容为实际数据的地址，也就是说它的索引和实际的数据是分开的，只不过是用索引指向了实际的数据，这种索引就是所谓的**非聚集索引**。

**因此，过程为：** MyISAM中索引检索的算法为首先按照B+Tree搜索算法搜索索引，如果指定的Key存在，则取出其data域的值，然后以data域的值为地址，根据data域的值去读取相应数据记录。



## InnoDB引擎的索引结构

也是B+Treee索引结构。Innodb的索引文件本身就是数据文件，即B+Tree的数据域存储的就是实际的数据，这种索引就是**聚集索引**。这个索引的key就是数据表的主键，因此InnoDB表数据文件本身就是主索引。

InnoDB的辅助索引数据域存储的也是相应记录主键的值而不是地址，所以当以辅助索引查找时，会先根据辅助索引找到主键，再根据主键索引找到实际的数据。**所以Innodb不建议使用过长的主键，否则会使辅助索引变得过大**。

**建议使用自增的字段作为主键，这样B+Tree的每一个结点都会被顺序的填满，而不会频繁的分裂调整，会有效的提升插入数据的效率。**
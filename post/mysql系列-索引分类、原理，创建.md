---
title:       "mysql系列-索引分类、原理，创建"
subtitle:    ""
description: "索引分类、原理，创建"
date:        2019-08-28
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["mysql系列", "索引分类、原理，创建"]
categories:  ["Tech" ]
---

[TOC]

**转载地址:<https://blog.csdn.net/jack__frost/article/details/72571540**>

# 索引的概述

## 什么是索引

**索引是一种特殊的文件(InnoDB数据表上的索引是表空间的一个组成部分)，它们包含着对数据表里所有记录的引用指针。**更通俗的说，数据库索引好比是一本书前面的目录，能加快数据库的查询速度。在没有索引的情况下，数据库会遍历全部数据后选择符合条件的；而有了相应的索引之后，数据库会直接在索引中查找符合条件的选项。

索引是一种特殊的文件(InnoDB数据表上的索引是表空间的一个组成部分)，它们包含着对数据表里所有记录的引用指针。更通俗的说，数据库索引好比是一本书前面的目录，能加快数据库的查询速度。在没有索引的情况下，数据库会遍历全部数据后选择符合条件的；而有了相应的索引之后，数据库会直接在索引中查找符合条件的选项。

## 索引的性质分类

索引分为聚簇索引和非聚簇索引两种，聚簇索引是按照数据存放的物理位置为顺序的，而非聚簇索引就不一样了；聚簇索引能提高多行检索的速度，而非聚簇索引对于单行的检索很快。

## 索引的优点

1. 通过创建唯一性索引，可以保证数据库表中每一行数据的唯一性。
2. 可以大大加快数据的检索速度，这也是创建索引的最主要的原因。
3. 可以加速表和表之间的连接，特别是在实现数据的参考完整性方面特别有意义。
4. 在使用分组和排序 子句进行数据检索时，同样可以显著减少查询中分组和排序的时间。
5. 通过使用索引，可以在查询的过程中，使用优化隐藏器，提高系统的性能。

## 索引的缺点

1. 创建索引和维护索引要耗费时间，这种时间随着数据量的增加而增加。
2. 索引需要占物理空间，除了数据表占数据空间之外，每一个索引还要占一定的物理空间，如果要建立聚簇索引，那么需要的空间就会更大。
3. 当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，这样就降低了数据的维护速度。

## 为什么需要索引

数据在磁盘上是以块的形式存储的。为确保对磁盘操作的原子性，访问数据的时候会一并访问所有数据块。磁盘上的这些数据块与链表类似，即它们都包含一个数据段和一个指针，指针指向下一个节点（数据块）的内存地址，而且它们都不需要连续存储（**即逻辑上相邻的数据块在物理上可以相隔很远**）。

鉴于很多记录只能做到按一个字段排序，所以要查询某个未经排序的字段，就需要使用线性查找，即要访问N/2个数据块，其中N指的是一个表所涵盖的所有数据块。如果该字段是非键字段（也就是说，不包含唯一值），那么就要搜索整个表空间，即要访问全部N个数据块。（在某些情况下，索引可以避免排序操作。）

然而，对于经过排序的字段，可以使用二分查找，因此只要访问log2 N个数据块。**同样，对于已经排过序的非键字段，只要找到更大的值，也就不用再搜索表中的其他数据块了。这样一来，性能就会有实质性的提升。**


# 索引的使用

## 创建索引

### CREATE TABLE 时创建索引

```sql
CREATE TABLE black_list (
	id int NOT NULL auto_increment ,
	black_user_id varchar(20) NULL DEFAULT NULL,
	user_id int DEFAULT 0,
	PRIMARY KEY (id),
  INDEX indexName (black_user_id(2))
);
```



### 使用ALTER TABLE命令去增加索引

ALTER TABLE用来创建普通索引、UNIQUE索引或PRIMARY KEY索引。

```sql
//标准语句：
ALTER TABLE table_name ADD INDEX index_name (column_list)//添加普通索引，索引值可出现多次。 
ALTER TABLE table_name ADD UNIQUE (column_list)//这条语句创建的索引的值必须是唯一的(除了NULL外，NULL可能会出现多次)。 
ALTER TABLE table_name ADD PRIMARY KEY (column_list)//该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。
ALTER TABLE table_name ADD FULLTEXT index_name(olumu_name);该语句指定了索引为FULLTEXT，用于全文索引。
```

其中table_name是要增加索引的表名，**column_list指出对哪些列进行索引，多列时各列之间用逗号分隔**。索引名index_name可自己命名，缺省时，MySQL将根据第一个索引列赋一个名称。另外，**ALTER TABLE允许在单个语句中更改多个表，因此可以在同时创建多个索引。**



### 使用CREATE INDEX命令创建

CREATE INDEX可对表增加普通索引或UNIQUE索引。

```sql
//标准语句：
CREATE INDEX index_name ON table_name (column_list)
CREATE UNIQUE INDEX index_name ON table_name (column_list)
//针对上述数据库：
CREATE INDEX classify_index  ON commodity_list (Classify_Description)
```

**注意：不能用CREATE INDEX语句创建PRIMARY KEY索引。**



## 删除索引

删除索引可以使用ALTER TABLE或DROP INDEX语句来实现。DROP INDEX可以在ALTER TABLE内部作为一条语句处理，其格式如下：

```sql
DROP INDEX [indexName] ON [table_name];
alter table [table_name] drop index [index_name] ;
alter table [table_name] drop primary key ;

//针对上述数据库
drop index classify_index on commodity_list ;
```

其中，在前面的两条语句中，都删除了table_name中的索引index_name。而在最后一条语句中，只在删除PRIMARY KEY索引中使用，**因为一个表只可能有一个PRIMARY KEY索引，因此不需要指定索引名。如果没有创建PRIMARY KEY索引，但表具有一个或多个UNIQUE索引，则MySQL将删除第一个UNIQUE索引。**

如果从表中删除某列，则索引会受影响。对于多列组合的索引，如果删除其中的某列，则该列也会从索引中删除。如果删除组成索引的所有列，则整个索引将被删除。



## 查看索引

```sql
SHOW INDEX FROM [table_name];
show keys from [table_name];
```

![Xnip2019-08-28_12-36-40](/img/Xnip2019-08-28_12-36-40.png)

1. Table：表的名称。

2. Non_unique：如果索引不能包括重复词，则为0。如果可以，则为1。

3. Key_name：索引的名称。

4. Seq_in_index：索引中的列序列号，从1开始。

5. Column_name：列名称。

6. Collation：列以什么方式存储在索引中。在MySQL中，有值‘A’（升序）或NULL（无分类）。

7. Sub_part：如果列只是被部分地编入索引，则为被编入索引的字符的数目。如果整列被编入索引，则为NULL

8. Packed：指示关键字如何被压缩。如果没有被压缩，则为NULL。

9. Null：如果列含有NULL，则含有YES。如果没有，则该列含有NO。

10. Index_type：用过的索引方法（BTREE, FULLTEXT, HASH, RTREE）

11. Comment：注释

12. Cardinality

    索引中唯一值的数目的估计值。通过运行ANALYZE TABLE或myisamchk -a可以更新。基数根据被存储为整数的统计数据来计数，所以即使对于小型表，该值也没有必要是精确的。基数越大，当进行联合时，MySQL使用该索引的机会就越大。



## 单列索引与多列索引

### 单列索引

就是平常的只索引一个一个的字段的方式。

```sql
//例子为name列的头10个字符创建一个索引：
CREATE TABLE test (
       name CHAR(200) NOT NULL,
       KEY index_name (name(10))
);
```

### 多列索引（也叫组合索引）

MySQL能在多个列上创建索引。一个索引可以由最多15个列组成。

多个单列索引与单个多列索引的查询效果不同，因为执行查询时，**MySQL只能使用一个索引**，会从多个单列索引中选择一个限制最为严格（获得结果集记录数最少）的索引。

#### 组合索引（多列索引）的原则

最左前缀：顾名思义，就是最左优先。在创建多列索引时，要根据业务需求，where子句中使用最频繁的一列放在最左边。



## 强制使用索引

有的时候MySQL优化器采取它认为合适的索引来检索sql语句，但是可能它所采用的索引并不是我们想要的。这时就可以采用force index来强制优化器使用我们制定的索引。**必要时可以使用force index来强制查询走某个索引**。

```sql
SELECT
  a.id as article_id ,
  a.title as title ,
  a.author_id as author_id 
from
  article a  FORCE INDEX(idx_createTime)
where
  a.create_time between '2020-03-22 03:00:00.003'
and '2020-04-22 03:00:00.003'
and a.status = 1
```



# 索引的基本原理

## 举例解析基本原理 

除了词典，生活中随处可见索引的例子，如火车站的车次表、图书的目录等。它们的原理都是一样的，通过不断的缩小想要获得数据的范围来筛选出最终想要的结果，同时把随机的事件变成顺序的事件，也就是我们总是通过同一种查找方式来锁定数据。

## SQL的应用场景会使用索引

数据库也是一样，但显然要复杂许多，因为不仅面临着等值查询，还有范围查询(>、<、between、in)、模糊查询(like)、并集查询(or)等等。数据库应该选择怎么样的方式来应对所有的问题呢？我们回想字典的例子，能不能把数据分成段，然后分段查询呢？最简单的如果1000条数据，1到100分成第一段，101到200分成第二段，201到300分成第三段…这样查第250条数据，只要找第三段就可以了，一下子去除了90%的无效数据。

## B+树搜索基本流程

**转载地址:<https://www.jianshu.com/p/43e6a7ac89c7>**

有一种数据结构就是考虑了上述种种需求，利用了种种原理设计出来的，就是`mysql`的索引所使用的数据结构: `B+`树。

![Xnip2019-08-28_12-51-53](/img/Xnip2019-08-28_12-51-53.png)

浅蓝色的块就是一个磁盘快，其中包含数据项和指针，如磁盘块1中包含数据项`17`和`35`,和指针 `P1,P2,P3`, `P1`指向所有小于17的数据所在的磁盘块，`P2`指向17和35之间的数据所在的磁盘快，`P3`指向大于35的磁盘块。真实的数据只存储在叶子节点中，非叶子节点只存储指引搜索方向的数据项和指针，不存储真实数据。如果要查询90，那么首先将磁盘快1加载到内存中，根据磁盘块1的数据项和指针确定90应该在磁盘块4中查找，于是从把磁盘快4加在到内存中，再次将搜索范围缩小至磁盘快11，此时返现磁盘快11已经是叶子节点，因此对磁盘块11中的数据做二分查找，最终返回查找结果，总计发生三次`IO`。实际上数据库实现时会把靠近树根的几层节点常驻内存，不需要发生`IO`操作，之后的三层`B+`树可以表示上百万的数据，如果上百万的数据查询只需要三次IO操作，性能提高将是巨大的。

# 索引分类



## 索引概念分类

### 聚集索引

#### 定义

该索引中键值的逻辑顺序决定了表中相应行的物理顺序。

聚集索引确定表中数据的物理顺序。聚集索引类似于电话簿，后者按姓氏排列数据。由于聚集索引规定数据在表中的物理存储顺序，因此一个表只能包含一个聚集索引。但该索引可以包含多个列（组合索引），就像电话簿按姓氏和名字进行组织一样。

#### 注意事项

定义聚集索引键时使用的列越少越好。



### 非聚集索引

#### 定义

数据存储在一个地方，索引存储在另一个地方，索引带有指针指向数据的存储位置。

非聚集索引中的项目按索引键值的顺序存储，而表中的信息按另一种顺序存储（这可以由聚集索引规定）。对于非聚集索引，可以为在表非聚集索引中查找数据时常用的每个列创建一个非聚集索引。有些书籍包含多个索引。例如，一本介绍园艺的书可能会包含一个植物通俗名称索引，和一个植物学名索引，因为这是读者查找信息的两种最常用的方法。

### 关于聚集索引以及非聚集索引的几个问题

1. **聚集索引的约束是唯一性，是否要求字段也是唯一的呢？** 

    一般我们指定一个表的主键，如果这个表之前没有聚集索引，同时建立主键时候没有强制指定使用非聚集索引，**SQL会默认在此字段上创建一个聚集索引，**而主键都是唯一的，所以理所当然的认为创建聚集索引的字段也需要唯一。

   

2. **主键就是聚集索引？？？**

   这样有时会对聚集索引的一种浪费。Innodb将通过主键聚集数据，如果没有定义主键，Innodb会选择第一个非空的唯一索引代替，如果没有非空唯一索引，Innodb会隐式定义一个6字节的rowid主键来作为聚集索引。innodb只聚集在同一个页面中的记录，包含相邻键值的页面可能会相距甚远。

   因为每个表中只能有一个聚集索引的规则，这使得聚集索引变得更加珍贵。

   

3. **是不是聚集索引就一定要比非聚集索引性能优呢？？？**

   如果想查询学分在60-90之间的学生的学分以及姓名，在学分上创建聚集索引是否是最优的呢？

   答：否。既然只输出两列，我们可以在学分以及学生姓名上创建联合非聚集索引（也就是多列索引），**此时的索引就形成了覆盖索引，即索引所存储的内容就是最终输出的数据，这两项限制直接可以定位出数据**，这种索引在比以学分为聚集索引做查询性能更好。

   

4. **在数据库中通过什么描述聚集索引与非聚集索引的？**

   索引是通过二叉树的形式进行描述的，我们可以这样区分聚集与非聚集索引的区别：聚集索引的叶节点就是最终的数据节点，而非聚集索引的叶节仍然是索引节点，但它有一个指向最终数据的指针。

   

5. **在主键是创建聚集索引的表在数据插入上为什么比主键上创建非聚集索引表速度要慢？**

   在有主键的表中插入数据行，由于有主键唯一性的约束，所以需要保证插入的数据没有重复。我们来比较下主键为聚集索引和非聚集索引的查找情况：聚集索引由于索引叶节点就是数据页，所以如果想检查主键的唯一性，需要遍历所有数据节点才行，但非聚集索引不同，由于非聚集索引上已经包含了主键值，所以查找主键唯一性，只需要遍历所有的索引页就行，这比遍历所有数据行减少了不少IO消耗。这就是为什么主键上创建非聚集索引比主键上创建聚集索引在插入数据时要快的真正原因。
   

## 索引具体分类



### 普通索引

基本的索引，它没有任何限制。

#### 创建方式

```sql
//标准语句：
ALTER TABLE table_name ADD INDEX index_name (column_list)
CREATE INDEX index_name ON table_name (column_list); 

//还有建表的时候创建亦可
CREATE TABLE table_name ( 
             ID INT NOT NULL, 
             column_listVARCHAR(16) NOT NULL,
             INDEX [index_name ] (column_list(length))
);  

```

如果是CHAR，VARCHAR类型，**length可以小于字段实际长度；如果是BLOB和TEXT类型，必须指定 length。**

例子：假如length为10，也就是索引这个字段的记录的前10个字符。



### 唯一索引

与前面的普通索引类似，不同的就是：MySQL数据库索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。

#### 创建方式

```sql
ALTER TABLE table_name ADD UNIQUE (column_list)
CREATE UNIQUE INDEX index_name ON table_name (column_list)

//还有建表时创建
CREATE TABLE table_name (
             ID INT NOT NULL, 
             column_list VARCHAR(16) NOT NULL, 
             UNIQUE [index_name ] (column_list(length)) 
 );  
```



### 主键索引

它是一种特殊的唯一索引，不允许有空值。一般是在建表的时候同时创建主键索引：

#### 创建方式

```sql
CREATE TABLE table_name ( 
             ID INT NOT NULL,
             [column] VARCHAR(16) NOT NULL,
             PRIMARY KEY(ID)  
 );  

```



### 全文索引：（FULLTEXT）

#### 定义

全文检索是对大数据文本进行索引，在建立的索引中对要查找的单词进行进行搜索，定位哪些文本数据包括要搜索的单词。因此，**全文检索的全部工作就是建立索引和在索引中搜索定位，所有的工作都是围绕这两个来进行的。**

#### 此索引关键

建立全文索引中有两项非常重要，一个是如何对文本进行分词，一是建立索引的数据结构。分词的方法基本上是二元分词法、最大匹配法和统计方法。索引的数据结构基本上采用倒排索引的结构。分词的好坏关系到查询的准确程度和生成的索引的大小。

#### 应用

FULLTEXT索引仅可用于 MyISAM 表；他们可以从CHAR、VARCHAR或TEXT列中作为CREATE TABLE语句的一部分被创建，或是随后使用ALTER TABLE 或CREATE INDEX被添加。

但是要注意：对于较大的数据集，将你的资料输入一个没有FULLTEXT索引的表中，然后创建索引，其速度比把资料输入现有FULLTEXT索引的速度更为快。不过切记对于大容量的数据表，生成全文索引是一个非常消耗时间非常消耗硬盘空间的做法。因为！！插入修改删除表的同时也要针对索引做一系列的处理。

#### 创建方式

```sql
//针对content做了全文索引：
CREATE TABLE `table` (
              `id` int(11) NOT NULL AUTO_INCREMENT ,
               `title` char(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL ,
               `content` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL ,
                PRIMARY KEY (`id`),
                FULLTEXT (content)
);

```

**注意：SQL使用全文索引的方法：首先必须是MyISAM的数据库引擎的数据表。（没有测试不知道其他引擎是否支持）**

```sql
SELECT * FROM article WHERE MATCH( content) AGAINST('想查询的字符串')
```



#### 注意

目前，使用MySQL自带的全文索引时，如果查询字符串的长度过短将无法得到期望的搜索结果。MySQL全文索引所能找到的词的默认最小长度为**4个字符**。另外，如果查询的字符串包含停止词，那么该停止词将会被忽略。

如果可能，请尽量先创建表并插入所有数据后再创建全文索引，而不要在创建表时就直接创建全文索引，因为前者比后者的全文索引效率要高。


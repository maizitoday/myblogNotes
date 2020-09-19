---
title:       "mysql系列-HEAP"
subtitle:    ""
description: "mysql内存表"
date:        2020-06-08
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["mysql系列","mysql内存表"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：http://www.zui88.com/blog/view-275.html**

# 概述

HEAP表是访问数据速度最快的MySQL表，他使用保存在内存中的散列索引。但如果MySQL或者服务器重新启动，表中数据将会丢失.

用法：如论坛的在线人数统计，这种表的数据应该是无关紧要的,就几个简单的字段,数据也不多,记录数怎么也不会超过1000吧,但是操作是最频繁的(基本用户的每次动作都要更新这个表).

# 如何创建内存表？

创建内存表非常的简单，只需注明 ENGINE= MEMORY 即可:

```sql
CREATE TABLE `tablename` ( `columnName` varchar(256) NOT NUL) ENGINE=MEMORY DEFAULT CHARSET=latin1 MAX_ROWS=100000000;
```

注意：
当内存表中的数据大于max_heap_table_size设定的容量大小时，mysql会转换超出的数据存储到磁盘上，因此这是性能就大打折扣了，所 以我们还需要根据我们的实际情况调整max_heap_table_size，例如在.cnf文件中[mysqld]的下面加入：

```properties
max_heap_table_size = 2048M
```

另外在建表语句中还可以通过MAX_ROWS来控制表的记录数

# 特性

内存表使用哈希散列索引把数据保存在内存中，因此具有极快的速度，适合缓存中小型数据库，但是使用上受到一些限制。

1、heap对所有用户的连接是可见的，这使得它非常适合做缓存。

2、仅适合使用的场合。heap不允许使用xxxTEXT和xxxBLOB数据类型；只允许使用=和&lt;=&gt;操作符来搜索记录 （不允许&lt;、&gt;、&lt;=或&gt;=）；不支持auto_increment；只允许对非空数据列进行 索引（not null）。
注：操作符 “&lt;=&gt;” 说明：NULL-safe equal.这个操作符和“=”操作符执行相同的比较操作，不过在两个操作码均为NULL时，其所得值为1而不为NULL，而当一个操作码为NULL时，其所得值为0而不为NULL。

3、一旦服务器重启，所有heap表数据丢失，但是heap表结构仍然存在，因为heap表结构是存放在实际数据库路径下的，不会自动删除。重启之后，heap将被清空，这时候对heap的查询结果都是空的。

4、如果heap是复制的某数据表，则复制之后所有主键、索引、自增等格式将不复存在，需要重新添加主键和索引，如果需要的话。

5、对于重启造成的数据丢失，有以下的解决办法：

```pro
a、在任何查询之前，执行一次简单的查询，判断heap表是否存在数据，如果不存在，则把数据重新写入，或者DROP表重新复制某张表。这需要多做一次查询。不过可以写成include文件，在需要用该heap表的页面随时调用，比较方便。

b、对于需要该heap表的页面，在该页面第一次且仅在第一次查询该表时，对数据集结果进行判断，如果结果为空，则需要重新写入数据。这样可以节省一次查询。

c、更好的办法是在mysql每次重新启动时自动写入数据到heap，但是需要配置服务器，过程比较复杂，通用性受到限制。
```



# 几个关键参数

```properties
max_heap_table_size  //设定的容量大小

ALTER TABLE tbl_name MAX_ROWS=   // 如果目前支持的行数到上限还不够用 可以把 my.conf 配置里面

AX_ROWS 依赖于 max_heap_table_size 设置
```


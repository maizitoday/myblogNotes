---
title:       "mysql系列-覆盖索引与回表"
subtitle:    ""
description: "索引的执行流程图片话，覆盖索引如何优化，回表是什么, count查询优化,分页优化,Extra列的NULL表示进行了回表查询"
date:        2020-09-19
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["mysql系列","索引的执行流程图片话，覆盖索引如何优化，回表是什么,count查询优化，分页优化,Extra列的NULL表示进行了回表查询"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://juejin.im/post/6844904062329028621   作者：张德Talk**



![Xnip2020-09-19_20-17-22](/img/Xnip2020-09-19_20-17-22.png)

# 两大类索引

```
使用的存储引擎：MySQL5.7 InnoDB
```



## 聚簇索引

```
* 如果表设置了主键，则主键就是聚簇索引
* 如果表没有主键，则会默认第一个NOT NULL，且唯一（UNIQUE）的列作为聚簇索引
* 以上都没有，则会默认创建一个隐藏的row_id作为聚簇索引
```

InnoDB的聚簇索引的叶子节点存储的是行记录（其实是页结构，一个页包含多行数据），InnoDB必须要有至少一个聚簇索引。

**由此可见，使用聚簇索引查询会很快，因为可以直接定位到行记录。**



## 普通索引

普通索引也叫二级索引，除聚簇索引外的索引，即非聚簇索引。

InnoDB的普通索引叶子节点存储的是主键（聚簇索引）的值，而MyISAM的普通索引存储的是记录指针。



# 示例

## 建表

```sql
mysql> create table user(
    -> id int(10) auto_increment,
    -> name varchar(30),
    -> age tinyint(4),
    -> primary key (id),
    -> index idx_age (age)
    -> )engine=innodb charset=utf8mb4;

```

id 字段是聚簇索引，age 字段是普通索引（二级索引）



## 填充数据

```sql
insert into user(name,age) values('张三',30);
insert into user(name,age) values('李四',20);
insert into user(name,age) values('王五',40);
insert into user(name,age) values('刘八',10);

mysql> select * from user;
+----+--------+------+
| id | name  | age |
+----+--------+------+
| 1 | 张三  |  30 |
| 2 | 李四  |  20 |
| 3 | 王五  |  40 |
| 4 | 刘八  |  10 |
+----+--------+------+

```



## 索引存储结构



### 聚簇索引（ClusteredIndex）

id 是主键，所以是聚簇索引，其叶子节点存储的是对应行记录的数据。

![Xnip2020-09-19_20-20-52](/img/Xnip2020-09-19_20-20-52.png)



### 普通索引（secondaryIndex）

age 是普通索引（二级索引），非聚簇索引，其叶子节点存储的是聚簇索引的的值

![Xnip2020-09-19_20-22-37](/img/Xnip2020-09-19_20-22-37.png)



### 查询

#### 聚簇索引查找过程

如果查询条件为主键（聚簇索引），则只需扫描一次B+树即可通过聚簇索引定位到要查找的行记录数据。

```sql
如：select * from user where id = 1;
```

![Xnip2020-09-19_20-24-52](/img/Xnip2020-09-19_20-24-52.png)



#### 普通索引查找过程

如果查询条件为普通索引（非聚簇索引），需要扫描两次B+树，第一次扫描通过普通索引定位到聚簇索引的值，然后第二次扫描通过聚簇索引的值定位到要查找的行记录数据。

```sql
 如：select * from user where age = 30;
```



##### 普通索引查找过程第一步

```
1. 先通过普通索引 age=30 定位到主键值 id=1
2. 再通过聚集索引 id=1 定位到行记录数据
```

![Xnip2020-09-19_20-27-32](/img/Xnip2020-09-19_20-27-32.png)

##### 普通索引查找过程第二步

![Xnip2020-09-19_20-28-07](/img/Xnip2020-09-19_20-28-07.png)



### 回表查询

先通过普通索引的值定位聚簇索引值，再通过聚簇索引的值定位行记录数据，需要扫描两次索引B+树，它的性能较扫一遍索引树更低。



## 索引覆盖

只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度更快。

```sql
例如：select id,age from user where age = 10;
```



### 如何实现覆盖索引

常见的方法是：将被查询的字段，建立到联合索引里去。

```sql
1、如实现：select id,age from user where age = 10;
```

explain分析：因为age是普通索引，使用到了age索引，通过一次扫描B+树即可查询到相应的结果，这样就实现了覆盖索引

![Xnip2020-09-19_20-32-03](/img/Xnip2020-09-19_20-32-03.png)



```sql
2、实现：select id,age,name from user where age = 10;
```

explain分析：age是普通索引，但name列不在索引树上，所以通过age索引在查询到id和age的值后，需要进行回表再查询name的值。**此时的Extra列的NULL表示进行了回表查询**。

![Xnip2020-09-19_20-33-28](/img/Xnip2020-09-19_20-33-28.png)

为了实现索引覆盖，需要建组合索引idx_age_name(age,name)

```sql
drop index idx_age on user;
create index idx_age_name on user(`age`,`name`);

```

explain分析：此时字段age和name是组合索引idx_age_name，查询的字段id、age、name的值刚刚都在索引树上，只需扫描一次组合索引B+树即可，这就是实现了索引覆盖，此时的Extra字段为Using index表示使用了索引覆盖。

![Xnip2020-09-19_20-34-29](/img/Xnip2020-09-19_20-34-29.png)



# 哪些场景适合使用索引覆盖来优化SQL



## 全表count查询优化

```sql
mysql> create table user(
    -> id int(10) auto_increment,
    -> name varchar(30),
    -> age tinyint(4),
    -> primary key (id),
    -> )engine=innodb charset=utf8mb4;

```

```sql
例如：select count(age) from user;
```

![Xnip2020-09-19_20-36-32](/img/Xnip2020-09-19_20-36-32.png)

使用索引覆盖优化：创建age字段索引

```sql
create index idx_age on user(age);
```

![Xnip2020-09-19_20-37-11](/img/Xnip2020-09-19_20-37-11.png)



## 列查询回表优化

前文在描述索引覆盖使用的例子就是

```sql
例如：select id,age,name from user where age = 10;
```

使用索引覆盖：建组合索引idx_age_name(age,name)即可



## 分页查询

```sql
例如：select id,age,name from user order by age limit 100,2;
```

因为name字段不是索引，所以在分页查询需要进行回表查询，此时Extra为Using filesort文件排序，查询性能低下。

![Xnip2020-09-19_20-38-45](/img/Xnip2020-09-19_20-38-45.png)

使用索引覆盖：建组合索引idx_age_name(age,name)

![Xnip2020-09-19_20-39-17](/img/Xnip2020-09-19_20-39-17.png)










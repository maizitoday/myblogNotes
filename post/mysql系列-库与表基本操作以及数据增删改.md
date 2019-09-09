title:       "mysql系列-库与表基本操作以及数据增删改"
subtitle:    ""
description: ""
date:        2019-08-26
author:      "麦子"
image:       "https://c.pxhere.com/images/e8/66/3d717bad8449de9495d9c9761d57-1423465.jpg!d"
tags:        ["mysql系列", "库与表基本操作","数据增删改"]
categories:  ["Tech" ]

[TOC]

**转载地址：<https://blog.csdn.net/jack__frost/article/details/71194208**>

# 操作数据操作语句优化认识

通常情况下，当访问某张表的时候，读取者首先必须获取该表的锁，如果有写入操作到达，那么写入者一直等待读取者完成操作（查询开始之后就不能中断，因此允许读取者完成操作）。当读取者完成对表的操作的时候，锁就会被解除。如果写入者正在等待的时候，另一个读取操作到达了，该读取操作也会被阻塞（block），因为默认的调度策略是写入者优先于读取者。当第一个读取者完成操作并解放锁后，写入者开始操作，并且直到该写入者完成操作，第二个读取者才开始操作。

通过LOCK TABLES和UNLOCK TABLES语句可以显式地获取或释放锁，但是在通常情况下，服务器的锁管理器会自动地在需要的时候获取锁，在不再需要的时候释放锁。获取的锁的类型依赖于客户端是写入还是读取操作。

**因为读取操作不会改变数据，因此没有理由让某个读取者阻止其它的读取者访问这张表。故读取锁可允许其它的客户端在同一时刻读取这张表。**

虽然通过锁机制，可以实现多线程同时对某个表进行操作，但当某个线程作更新操作时，首先要获得独占的访问权。在更新的过程中，所有其它想要访问这个表的线程必须要等到其更新完成为止。此时就会导致锁竞争的问题，从而导致用户等待时间的延长。**这也就是NOSQL出现的原因，在高并发的情况下， 读取数据MySQL太慢。** 

**因此：要提高MySQL的更新/插入效率，应首先考虑降低锁的竞争，减少写操作的等待时间。** 



# 表增删查改

## INSERT语句

### 用法

```sql
INSERT [INTO] 表名 [(字段列表)] VALUES (值列表)[, (值列表), …]
```

如果要插入的值列表包含所有字段并且顺序一致，则可以省略字段列表。

可同时插入多条数据记录！

```sql
INSERT INTO score (change_type,score,user_id) VALUES ('吃饭',10,1),('喝茶',10,1),('喝茶',10,1);   
```



### 优化

#### 批量插入

例如说，如果有需要插入100000条数据，那么就需要有100000条insert语句，每一句都需要提交到关系引擎那里去解析，优化，然后才能够到达存储引擎做真的插入工作。上述所说的同时插入多条就是一种优化。（经测试，大概10条同时插入是最高效的）

#### 顺序主键策略

采用顺序主键策略（例如自增主键，或者修改业务逻辑，让插入的记录尽可能顺序主键）

#### replace 语句代替insert

考虑使用replace 语句代替insert语句



## DELETE语句

### 用法

```sql
DELETE FROM 表名[ 删除条件子句]
```



### truncate table和delete语句

1. truncate table速度要更快一些，但truncate删除后不记录mysql日志，不可以恢复数据。

2. 如果没有外键关联，innodb执行truncate是先drop table(原始表),再创建一个跟原始表一样空表,速度要远远快于delete逐条删除行记录。

3. truncate table 能重新利用释放的硬盘空间。truncate table 能重新利用释放的硬盘空间,在InnoDB Plugin中，truncate table为自动回收，如果不是用InnoDB Plugin（存储引擎）,那么需要使用optimize table来优化表，释放空间。**truncate table删除表后，optimize table尤其重要，特别是大数据数据库，表空间可以得到释放！**

4. 表有外键关联，truncate table删除表数据为逐行删除

   

### 注意

一个大的 DELETE 或 INSERT 操作，要非常小心，因为这两个操作是会锁表的，表一锁住，其他操作就进不来了。因此，我们要交给DBA去拆分，重整数据库策略，比如限制处理1000条。

**MySQL官方手 册得知删除数据的速度和创建的索引数量是成正比的**。所以在超大型数据库中，删除时处理好索引关系非常重要。推荐的折中方法：在删除数据之前删除这那几个索引，然后删除其中无用 数据，删除完成后重新创建索引。



## UPDATE语句

### 用法

```
UPDATE 表名 SET 字段名=新值[, 字段名=新值] [更新条件]
```

### 优化

#### 更新多条记录

```sql
Update score  
  SET change_type = CASE id  
    WHEN 1 THEN 'value1'  
    WHEN 2 THEN 'value2'  
    WHEN 3 THEN 'value3'  
  END  
WHERE id IN (1,2,3)  
```



#### 更新多条记录的多个值

```sql
Update score  
  SET change_type = CASE id  
    WHEN 1 THEN 'value1'  
    WHEN 2 THEN 'value2'  
    WHEN 3 THEN 'value3'  
  END , 
    score = CASE id
  	 WHEN 1 THEN 1
    WHEN 2 THEN 2
    WHEN 3 THEN 3
  END
WHERE id IN (1,2,3)  
```



#### 其他优化

1. 尽量不要修改主键字段。
2. 当修改VARCHAR型字段时，尽量使用相同长度内容的值代替。
3. 尽量最小化对于含有UPDATE触发器的表的UPDATE操作。
4. 避免UPDATE将要复制到其他数据库的列。
5. 避免UPDATE建有很多索引的列。
6. 避免UPDATE在WHERE子句条件中的列。

### 

## REPLACE语句

例如：如果一个表在一个字段上建立了唯一索引，当向这个表中使用已经存在的键值插入一条记录，将会抛出一个主键冲突的错误。如果我们想用新记录的值来覆盖原来的记录值时，就可以使用REPLACE语句。

使用REPLACE插入记录时，如果记录不重复（或往表里插新记录），REPLACE功能与INSERT一样，如果存在重复记录，REPLACE就使用新记录的值来替换原来的记录值。使用REPLACE的最大好处就是可以将DELETE和INSERT合二为一，形成一个原子操作。这样就可以不必考虑同时使用DELETE和INSERT时添加事务等复杂操作了。

在使用REPLACE时，表中必须有唯一有一个PRIMARY KEY或UNIQUE索引，否则，使用一个REPLACE语句没有意义。

### 用法

此语句的作用是向表table中插入3条记录。如果主键id为1或2不存在就相当于插入语句。

```sql
//含义一：与普通INSERT一样功能
REPLACE INTO score (change_type,score,user_id) VALUES ('吃饭',10,1),('喝茶',10,1),('喝茶',10,1);  
//含义二：找到第一条记录，用后面的值进行替换
REPLACE INTO score (id,change_type,score,user_id) VALUES (1,'吃饭',10,1)

```



#### replace(object, search, replace)

把object中出现search的全部替换为replace。

```sql
//用法一：并不是修改数据，而只是单纯做局部替换数据返还而已。
SELECT REPLACE('喝茶','茶','喝')
//结果：  喝喝
```



#### 修改表数据

修改表数据啦，对应下面就是，根据change_type字段找到做任务的数据，用bb来替换，

```sql
UPDATE score SET change_type=REPLACE(change_type,'做任务','bb')
```



### UPDATE和REPLACE的区别

1. UPDATE在没有匹配记录时什么都不做，而REPLACE在有重复记录时更新，在没有重复记录时插入。
2. UPDATE可以选择性地更新记录的一部分字段。而REPLACE在发现有重复记录时就将这条记录彻底删除，再插入新的记录。也就是说，将所有的字段都更新了



# 库的基本操作

## 查看所有数据库以及使用数据库

```sql
show databases;

use table;
```



## 查看当前数据库

```sql
 select database();
```



## 显示当前时间、用户名、数据库版本

```sql
select now(), user(), version();
```



## 创建库

```sql
create database[ if not exists] 数据库名 数据库选项
    数据库选项：
        CHARACTER SET charset_name
        COLLATE collation_name		
```



## 查看当前库信息

```mysql
show create database 数据库名	
```



## 修改库的选项信息

```sql
alter database 库名 选项信息
```



## 删除库

```sql
drop database[ if exists] 数据库名
        同时删除该数据库相关的目录及其目录内容
```



# 表的基本操作

## 创建表

```sql
 create [temporary] table[ if not exists] [库名.]表名 ( 表的结构定义 )[ 表选项]
        每个字段必须有数据类型
        最后一个字段后不能有逗号
        temporary 临时表，会话结束时表自动消失
        对于字段的定义：
            字段名 数据类型 [NOT NULL | NULL] [DEFAULT default_value] [AUTO_INCREMENT] [UNIQUE [KEY] | [PRIMARY] KEY] [COMMENT 'string']

```



## 表的选项

### 删除列

```sql
ALTER TABLE 【表名字】 DROP 【列名称】
```

### 增加列

```sql
ALTER TABLE 【表名字】 ADD 【列名称】 INT NOT NULL COMMENT ‘注释说明’
```

### 修改列的类型信息

```sql
ALTER TABLE 【表名字】 CHANGE 【列名称】【新列名称（这里可以用和原来列同名即可）】 BIGINT NOT NULL COMMENT ‘注释说明’
```

### 重命名列

```sql
ALTER TABLE 【表名字】 CHANGE 【列名称】【新列名称】 BIGINT NOT NULL COMMENT ‘注释说明’
```

### 重命名表

```sql
ALTER TABLE 【表名字】 RENAME 【表新名字】
```

### 删除表中主键

```sql
Alter TABLE 【表名字】 drop primary key
```

### 添加主键

```sql
ALTER TABLE sj_resource_charges ADD CONSTRAINT PK_SJ_RESOURCE_CHARGES PRIMARY KEY (resid,resfromid)
```

### 添加索引

```sql
ALTER TABLE sj_resource_charges add index INDEX_NAME (name);
```

### 添加唯一限制条件索引

```sql
ALTER TABLE sj_resource_charges add unique emp_name2(cardnumber);
```

### 删除索引

```sql
Alter table tablename drop index emp_name;
```



## 查看所有表

```sql
SHOW TABLES；
```



## 查看表机构

```sql
 SHOW CREATE TABLE 表名    （信息更详细）
    DESC 表名 / DESCRIBE 表名 / EXPLAIN 表名 / SHOW COLUMNS FROM 表名 [LIKE 'PATTERN']
    SHOW TABLE STATUS [FROM db_name] [LIKE 'pattern']

```



## 修改表：对表进行重命名

```sql
RENAME TABLE 原表名 TO 新表名
        RENAME TABLE 原表名 TO 库名.表名    （可将表移动到另一个数据库）
        -- RENAME可以交换两个表名
```



## 删除表

```sql
DROP TABLE[ IF EXISTS] 表名
```



## 清空表数据

```sql
TRUNCATE [TABLE] 表名
```



## 复制表结构

```sql
CREATE TABLE 表名 [AS] SELECT * FROM 要复制的表名
```



## 添加自增长

```sql
alter table tablename modify id int(11)  auto_increment;
```


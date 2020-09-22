---
title:       "mysql系列-索引设计和查询优化"
subtitle:    ""
description: "索引建立原则，索引注意点，什么时候索引会生效，索引优化，索引类型"
date:        2020-09-19
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["mysql系列", "索引建立原则，索引注意点，什么时候索引会生效，索引优化，索引类型"]
categories:  ["Tech" ]
---

[TOC]



**转载地址: <https://blog.csdn.net/jack__frost/article/details/72571540**>**

# 索引设计优化



## 适合使用索引的场景

1. 主键自动创建唯一索引
2. 频繁作为查询条件的字段
3. 查询中与其他表关联的字段
4. 查询中排序的字段
5. 查询中统计或分组字段



## 不适合使用索引的场景

1. 频繁更新的字段
2. where 条件中用不到的字段
3. 表记录太少
4. 经常增删改的表
5. 字段的值的差异性不大或重复性高



## 索引建立的几大原则

#### 1. 最左前缀匹配原则

非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。

#### 2. 尽量选择区分度高的列作为索引

表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1，而一些状态、性别字段可能在大数据面前区分度就是0，那可能有人会问，这个比例有什么经验值吗？使用场景不同，这个值也很难确定，一般需要join的字段我们都要求是0.1以上，即平均1条扫描10条记录。

#### 3. 索引列不能参与计算，保持列“干净”

比如from_unixtime(create_time) = ’2014-05-29’就不能使用到索引，原因很简单，**b+树中存的都是数据表中的字段值**，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大。所以语句应该写成create_time = unix_timestamp(’2014-05-29’);

#### 4. 尽量的扩展索引，不要新建索引。

比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。

#### 5. 定义有外键的数据列一定要建立索引。

#### 6. 对于那些查询中很少涉及的列，重复值比较多的列不要建立索引。

#### 7. 对于定义为text、image和bit的数据类型的列不要建立索引。

#### 8. 对于经常存取的列避免建立索引，因为索引字段的存取都要消耗很大性能。





## 索引和查询优化

#### 1.  JOIN,WHERE判断和ORDER BY排序的字段上建立索引

一般说来，索引应建立在那些将用于JOIN,WHERE判断和ORDER BY排序的字段上。尽量不要对数据库中某个含有大量重复的值的字段建立索引。对于一个ENUM类型的字段来说，出现大量重复值是很有可能的情况。



#### 2.  避免在 where 子句中对字段进行 null 值判断

应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行**全表扫描**。如：

```sql
select id from t where num is null

```

备注、描述、评论之类的可以设置为 NULL，其他的，最好不要使用NULL。
不要以为 NULL 不需要空间，比如：char(100) 型，在字段建立时，空间就固定了， 不管是否插入值（NULL也包含在内），都是占用 100个字符的空间的，
如果是varchar这样的变长字段， null 不占用空间。
可以在num上设置默认值0，确保表中num列没有null值，然后这样查询：

```sql
select id from t where num = 0;
```



#### 4.  应尽量避免在 where 子句中使用 != 或 <> 操作符，否则将引擎放弃使用索引而进行全表扫描。



#### 5.  应尽量避免在 where 子句中使用 or 来连接条件

应尽量避免在 where 子句中使用 or 来连接条件，如果一个字段有索引，一个字段没有索引，将导致引擎放弃使用索引而进行全表扫描。如：

```sql
select id from t where num=10 or Name = 'fuzhu'	

```

可以这样查询，充分利用索引：

```sql
select id from t where num = 10
union all
select id from t where Name = 'fuzhu'

```



#### 6.  in 和 not in 也要慎用，否则会导致全表扫描。

而且负向查询（not , not in, not like, <>, != ,!>,!< ） 不会使用索引

```sql
select id from t where num in(1,2,3)

```

对于连续的数值，能用 between 就不要用 in 了：

```sql
select id from t where num between 1 and 3

```

很多时候用 exists 代替 in 是一个好的选择，**当然exists也不跑索引**。

```sql
select num from a where num in(select num from b)

```



#### 7.  %aaa%”模糊查询也将导致全表扫描

```sql
select id from t where name like ‘%abc%’

```

一般情况下不鼓励使用like操作，如果非使用不可，如何使用也是一个问题。like “%aaa%” 不会使用索引，而like “aaa%”可以使用索引。**若要提高效率，可以考虑全文检索。**

#### a. like %keyword 索引失效，使用全表扫描。

#### b. like keyword% 索引有效。

#### c. like %keyword% 索引失效，



#### 8.  如果在 where 子句中使用参数，也会导致全表扫描。

因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；
它必须在编译时进行选择。然 而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描：

```sql
select id from t where num = @num
```


可以改为强制查询使用索引：

```sql
 select id from t with(index(索引名)) where num = @num； 
```



#### 8. 应尽量避免在 where 子句中对字段进行表达式操作，

这将导致引擎放弃使用索引而进行全表扫描。如：

```sql
select id from t where num/2 = 100
```

可以改为:

```sql
select id from t where num = 100*2；
```



#### 9.  应尽量避免在where子句中对字段进行函数操作。

这将导致引擎放弃使用索引而进行全表扫描

```sql
select id from t where substring(name,1,3) = ’abc’       //name以abc开头的id
select id from t where datediff(day,createdate,’2005-11-30′) = 0  -–‘2005-11-30’    //生成的id

```

**应改为:**

```sql
select id from t where name like 'abc%'
select id from t where createdate >= '2005-11-30' and createdate < '2005-12-1'
```



#### 10.  where 子句中的“=”左边进行函数、算术运算或其他表达式运算无法正确使用索引。



#### 11.  索引字段作为条件时，如果该索引是多列索引，需要注意

在使用索引字段作为条件时，如果该索引是复合索引（多列索引），那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。



#### 11. select count(*) from table；全表扫描

这样不带任何条件的count会引起全表扫描，并且没有任何业务意义，是一定要杜绝的；



#### 12.  索引并不是越多越好

索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，**因为 insert 或 update 时有可能会重建索引**，所以怎样建索引需要慎重考虑，视具体情况而定。**一个表的索引数最好不要超过6个**，若太多则应考虑一些不常使用到的列上建的索引是否有必要。



#### 13.  尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理



#### 14.  尽量使用数字型字段

尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。

这是因为引擎在处理查询和连 接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了；



#### 15. 任何地方都不要使用 select * from t ，

 用具体的字段列表代替“*”，不要返回用不到的任何字段；



#### 16. 尽量避免使用游标，因为游标的效率较差

如果游标操作的数据超过1万行，那么就应该考虑改写；



#### 17. 优化limit

比如：

```sql
select id from t order by id limit 99999,10
```

原语句虽然使用了id索引，但是相当于从第一行定位到99999行再去扫描后10行，相当于扫描全表
如果改为

```sql
select id from t where id>=100000 order by id limit 10
```

则直接定位到100000查找



#### 18. 利用LIMIT 1取得唯一行

当你要查询一张表是，你知道自己只需要看一行。在这种情况下，增加一个LIMIT 1会令你的查询更加有效。

这样数据库引擎发现只有1后将停止扫描，而不是去扫描整个表或索引。

不过如果主键做了索引后，直接查询 id = 1 ，这种形式也是很快的。 



#### 19. <及>操作

大于或小于一般情况不用调整，因为它有索引就会采用索引查找，但有的情况下可以对它进行优化。如一个表有100万记录，那么执行>2与>=3的效果就有很大区别了。

```sql
（低效）select * from [emp] where [deptno]>2
（高效）select * from [emp] where [deptno]>=3
```



#### 20. 在Join表的时候使用相当类型的例，并将其索引

如果你的应用程序有很多 JOIN 查询，你应该确认两个表中Join的字段是被建过索引的。这样，MySQL内部会启动为你优化Join的SQL语句的机制。而且，这些被用来Join的字段，应该是相同的类型的。

例如：如果你要把 DECIMAL 字段和一个 INT 字段Join在一起，MySQL就无法使用它们的索引。对于那些STRING类型，还需要有相同的字符集才行。（两个表的字符集有可能不一样）

```sql
SELECT company_name FROM users
  LEFT JOIN companies ON (users.state = companies.state)
   WHERE users.id = $user_id"
```

**两个 state 字段应该是被建过索引的，而且应该是相当的类型，相同的字符集**



#### 21. 使用分页语句优化

limit start , count 或者条件 where子句。有什么可限制的条件尽量加上，查一条就limit一条。做到不多拿不乱拿。对limit的使用再优化，利用自增主键，避免offset的使用（演示在积分表score，商品表设计得不太好），约是上面方法的1/3时间。

```sql
select * from school.student limit 0,1;  
```

```sql
select * from school.student where id > 1 limit 2; 
```



#### 22.  如果是有序的查询，可使用**ORDER BY**

```sql
select * from score  WHERE id>0  ORDER BY score ASC  LIMIT 10000;
```



#### 23. 开启查询缓存

大多数的MySQL服务器都开启了查询缓存。这是提高性最有效的方法之一。当有很多相同的查询被执行了多次的时候，这些查询结果会被放到一个缓存中，这样，后续的相同的查询就不用操作表而直接访问缓存结果了。

**命中缓存条件**

1. 缓存存在一个hash表中,通过查询SQL,查询数据库,客户端协议等作为key.在判断是否命中前,MySQL不会解析SQL,而是直接使用SQL去查询缓存,**SQL**任何字符上的不同,如空格,注释,都会导致缓存不命中**.**
2. 如果查询中有不确定数据,例如**CURRENT_DATE**()和**NOW**()函数,那么查询完毕后则不会被缓存.所以,包含不确定数据的查询是肯定不会找到可用缓存的



#### 24.  超过三个表禁止 join。

需要 join 的字段，数据类型必须绝对一致；多表关联查询时， 保证被关联的字段需要有索引。



#### 25.  如果有 order by 的场景，请注意利用索引的有序性。

order by 最后的字段是组合 索引的一部分，**并且放在索引组合顺序的最后**，避免出现 file_sort 的情况，影响查询性能。



#### 26.  不要使用 count(列名)或 count(常量)来替代 count()，

count()是 SQL92 定义的标准统计行数的语法，跟数据库无关，跟 NULL 和非 NULL 无关。 说明：count(\*)会统计值为 NULL 的行，而 count(列名)不会统计此列为 NULL 值的行。



#### 27.  count(distinct col) 计算该列除 NULL 之外的不重复行数，

注意 count(di col1, col2) 如果其中一列全为 NULL，那么即使另一列有不同的值，也返回为 0。



#### 28.  当某一列的值全是 NULL 时，count(col)的返回结果为 0，

但 sum(col)的返回结果为NULL，因此使用 sum()时需注意 NPE 问题。



#### 29.  使用 ISNULL()来判断是否为 NULL 值。



#### 30.  禁止使用存储过程，存储过程难以调试和扩展，更没有移植性。





## 联合索引问题优化

**转载地址：https://zhuanlan.zhihu.com/p/143464808**

细看《mysql系列-覆盖索引与回表》博客，对于他们的定义。

联合索引其实有两个作用：



#### 1.充分利用where条件，缩小范围

例如我们需要查询以下语句：

```sql
SELECT * FROM test WHERE a = 1 AND b = 2
```

如果对字段a建立单列索引，对b建立单列索引，那么在查询时，只能选择走索引a，查询所有a=1的主键id，然后进行回表，在回表的过程中，在聚集索引中读取每一行数据，然后过滤出b = 2结果集，或者走索引b，也是这样的过程。 如果对a，b建立了联合索引(a,b),那么在查询时，直接在联合索引中先查到a=1的节点，然后根据b=2继续往下查，查出符合条件的结果集，进行回表。



#### 2.避免回表(此时也叫覆盖索引)

这种情况就是假如我们只查询某几个常用字段，例如查询a和b如下：

```sql
SELECT a,b FROM test WHERE a = 1 AND b = 2
```

对字段a建立单列索引，对b建立单列索引就需要像上面所说的，查到符合条件的主键id集合后需要去聚集索引下回表查询，但是如果我们要查询的字段本身在联合索引中就都包含了，那么就不用回表了。



#### 3.减少需要回表的数据的行数

这种情况就是假如我们需要查询a>1并且b=2的数据

```sql
SELECT * FROM test WHERE a > 1 AND b = 2
```

如果建立的是单列索引a，那么在查询时会在单列索引a中把a>1的主键id全部查找出来然后进行回表。 如果建立的是联合索引(a,b),基于最左前缀匹配原则，因为a的查询条件是一个范围查找(=或者in之外的查询条件都是范围查找)，这样虽然在联合索引中查询时只能命中索引a的部分，b的部分命中不了，只能根据a>1进行查询，但是由于联合索引中每个叶子节点包含b的信息，在查询出所有a>1的主键id时，也会对b=2进行筛选，这样需要回表的主键id就只有a>1并且b=2这部分了，所以回表的数据量会变小。



#### 示例01

我们业务中碰到的就是第3种情况，我们的业务SQL本来更加复杂，还会join其他表，但是由于优化的瓶颈在于建立联合索引，所以进行了一些简化，下面是简化后的SQL：

```sql
SELECT
  a.id as article_id ,
  a.title as title ,
  a.author_id as author_id 
from
  article a
where
  a.create_time between '2020-03-29 03:00:00.003'
and '2020-04-29 03:00:00.003'
and a.status = 1
```

我们的需求其实就是从article表中查询出最近一个月，status为1的文章，我们本来就是针对create_time建了单列索引，结果在慢查询日志中发现了这条语句，查询时间需要0.91s左右，所以开始尝试着进行优化。

为了便于测试，我们在表中分别对create_time建立了单列索引create_time，对(create_time,status)建立联合索引idx_createTime_status。

强制使用idx_createTime进行查询

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

强制使用idx_createTime_status进行查询（即使不强制也是会选择这个索引）

```sql
SELECT
  a.id as article_id ,
  a.title as title ,
  a.author_id as author_id 
from
  article a  FORCE INDEX(idx_createTime_status)
where
  a.create_time between '2020-03-22 03:00:00.003'
and '2020-04-22 03:00:00.003'
and a.status = 1
```

**优化结果：**

优化前使用idx_createTime单列索引，查询时间为0.91s

优化前使用idx_createTime_status联合索引，查询时间为0.21s



#### 示例02

![5-19-1.png](/img/5-19-1.png)

![5-19-2.png](/img/5-19-2.png)



##### 什么是离散度

转载：https://www.cnblogs.com/toby/archive/2012/11/09/2763151.html

**mysql**数据中离散程度的大小的判定方法

离散程度的大小的判定方法：计算出数据库中记录不重复数量

```sql
select count(distinct row1),count(distinct row2) from table;
```

count()值大的就离散度高。 离散度大的列放在联合索引的前面。














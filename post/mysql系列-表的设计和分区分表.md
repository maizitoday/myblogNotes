---
title:       "mysql系列-表设计和分区分表"
subtitle:    ""
description: "表设计注意点，表数据类型，分区分表，表命名规则，数据库设计常见表"
date:        2019-08-28
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["mysql系列", "表设计注意点，表数据类型，分区分表，表命名规则，数据库设计常见表"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：<https://blog.csdn.net/jack__frost/article/details/72672252**>

# 单表设计与优化

## 设计规范化表，消除数据冗余

数据库范式是确保数据库结构合理，满足各种查询需要、避免数据库操作异常的数据库设计方式。满足范式要求的表，称为规范化表，范式产生于20世纪70年代初，一般表设计满足前三范式就可以，在这里简单介绍一下**前三范式**。

### 第一范式

属性(字段)的原子性约束，要求属性具有原子性，不可再分割；

### 第二范式

记录的惟一性约束，要求记录有惟一标识，每条记录需要有一个属性来做为实体的唯一标识，即每列都要和主键相关。

### 第三范式

属性(字段)冗余性的约束，即任何字段不能由其他字段派生出来，在通俗点就是：主键没有直接关系的数据列必须消除(消除的办法就是再创建一个表来存放他们，当然外键除外)。即：确保每列都和主键列直接相关,而不是间接相关。

如果数据库设计达到了完全的标准化，则把所有的表通过关键字连接在一起时，不会出现任何数据的复本(repetition)。标准化的优点是明显的，它避免了数据冗余，自然就节省了空间，也对数据的一致性(consistency)提供了根本的保障，杜绝了数据不一致的现象，同时也提高了效率。



## 一个字节可以存储多大的数字？

一个字节有8个位，每个位有2种状态， 1和0， 于是，

如果将首位作为符号位  即1为负数 0为整数， 则一字节存的最小数为 11111111（-128），最大数为 01111111（127）

如果没有符号位则   最小数为00000000（十进制0） 最大数为11111111（十进制255）



## tinyint(1) vs tinyint(4) 有什么区别

转载地址：https://my.oschina.net/DavidRicardo/blog/869169

tinyint后面的括号带的数字,以后称之为M,和存贮的值没有任何关系,只是在某些情况下和显示的宽度有关系。

最后,我给出了我的建议,那就是M其实没用,tinyint默认是4,其余的也有默认值,以后程序开发中,涉及整形数字的M时,可以不必纠结,直接忽略,最后使用数据库默认的M值即可

**说明：详情看上面转载文章。** 





## unsigned是什么意思?

转载地址：https://www.cnblogs.com/qiantuwuliang/archive/2009/11/22/1608157.html

unsigned, zerofill    既为非负数，用此类型可以增加数据长度，   

**unsigned**   既为非负数，用此类型可以增加数据长度!

例如如果    tinyint最大是127，那    tinyint    unsigned    最大   就可以到    127 * 2





## 优化(以使用正确字段类型最明显)

### 字段类型

#### 数字类型

##### 整型

| type      | Storage | Minumun Value        | Maximum Value        |
| --------- | ------- | -------------------- | -------------------- |
|           | (Bytes) | (Signed/Unsigned)    | (Signed/Unsigned)    |
| TINYINT   | 1       | -128                 | 127                  |
|           |         | 0                    | 255                  |
| SMALLINT  | 2       | -32768               | 32767                |
|           |         | 0                    | 65535                |
| MEDIUMINT | 3       | -8388608             | 8388607              |
|           |         | 0                    | 16777215             |
| INT       | 4       | -2147483648          | 2147483647           |
|           |         | 0                    | 4294967295           |
| BIGINT    | 8       | -9223372036854775808 | 9223372036854775807  |
|           |         | 0                    | 18446744073709551615 |

##### 浮点型

| 属性         | 存储空间 | 精度   | 精确性        | 说明                           |
| ------------ | -------- | ------ | ------------- | ------------------------------ |
| FLOAT(M, D)  | 4 bytes  | 单精度 | 非精确        | 单精度浮点型，m总个数，d小数位 |
| DOUBLE(M, D) | 8 bytes  | 双精度 | 比Float精度高 | 双精度浮点型，m总个数，d小数位 |

##### 定点数类型

| 定点数类型 | 字节数 | 最小值 ~ 最大值                                              |
| ---------- | ------ | ------------------------------------------------------------ |
| dec(m,d)   | m+2    | 最大取值范围与double相同，给定decimal的有效值取值范围由m和d决定 |



##### 选择数字类型总结

###### 1.  关于浮点数与定点数问题

浮点数相对于定点数的优点是在长度一定的情况下，浮点数能够表示更大的数据范围；它的缺点是会引起**精度问题**。

使用时我们要注意

1. 浮点数存在误差问题；

2. 对货币等对精度敏感的数据，应该用定点数表示或存储；

3. 编程中，如果用到浮点数，要特别注意误差问题，并尽量避免做浮点数比较；

4. 要注意浮点数中一些特殊值的处理。

###### 2.  不到不要使用DOUBLE，不仅仅只是存储长度的问题，同时还会存在精确性的问题。

###### 3.  固定精度的小数，也不建议使用DECIMAL

建议乘以固定倍数转换成整数存储，可以大大节省存储空间，且不会带来任何附加维护成本。整数只有1个字节存储， 但是这个也要看你实际项目情况。 

###### 4.  区分开 TINYINT / INT / BIGINT 

因为三者所占用的存储空间也有很大的差别。

###### 5.  对于整型数值，mysql支持在类型名称后面的小括号内指定显示宽度

例如int(5)表示当数值宽度小于5位时候在数值前面填满宽度，一般配合zerofill属性使用。如果一个列指定为zerofill,则MySQL自动为该列添加unsigned属性。**反正他就占用一个字节**

###### 6.  在数据量较大时、建议把实数类型转为整数类型。

原因很简单：1. 浮点不精确；2.定点计算代价昂贵。例如：要存放财务数据精确到万分之一、则可以把所有金额乘以一百万、然后存在BIGINT下。



### 时间类型

| 类型      | 字节   | 例                   | 精确性             |
| --------- | ------ | -------------------- | ------------------ |
| DATE      | 三字节 | 2015-05-01           | 精确到年月日       |
| TIME      | 三字节 | 11:12:00             | 精确到时分秒       |
| DATETIME  | 八字节 | 2015-05-01 11::12:00 | 精确到年月日时分秒 |
| TIMESTAMP | 四字节 | 2015-05-01 11::12:00 | 精确到年月日时分秒 |

#### 选择时间类型总结

##### 1.  TIMESTAMP`会根据系统时区进行转换，`DATETIME`则不会

##### 2.  尽量使用TIMESTAMP类型

因为其存储空间只需要 DATETIME 类型的一半。对于只需要精确到某一天的数据类型，建议使用DATE类型，因为他的存储空间只需要3个字节，比TIMESTAMP还少。

##### 3.  根据实际需要选择能够满足应用的最小存储日期类型





### 字符串类型

| 类型    | 单位 | 最大                        | 特性                         |
| ------- | ---- | --------------------------- | ---------------------------- |
| CHAR    | 字符 | 最大为255字符               | 存储定长，容易造成空间的浪费 |
| VARCHAR | 字符 | 可以超过255个字符           | 存储变长，节省存储空间       |
| TEXT    | 字节 | 总大小为65535字节，约为64KB | -                            |

#### 选择字符类型总结

##### 1.  varchar的长度？

MySQL的文档，其中对varchar字段类型这样描述：varchar(m) 变长字符串。m 表示最大列长度。m的范围是0到65,535。(VARCHAR的最大实际长度由最长的行的大小和使用的字符集确定，最大有效长度是65,532字节）。

mysql varchar(50) 不管中文 还是英文 都是存50个的，但是一个表中所有varchar字段的总长度跟编码有关，如果是utf-8，那么大概65535/3，如果是gbk，那么大概65535/2.

##### 2. 存储限制？编码长度限制？行长度限制？超出了？

针对第一个问题：varchar 字段是将实际内容单独存储在聚簇索引之外，实际存储从第二个字节开始，接着要用1到2个字节表示实际长度（长度超过255时需要2个字节），因此最大长度不能超过65535。

针对第二个问题：字符类型若为gbk，每个字符最多占2个字节。字符类型若为utf8，每个字符最多占3个字节。

针对第三个问题：导致实际应用中varchar长度限制的是一个行定义的长度。 MySQL要求一个行的定义长度不能超过65535。若定义的表长度超过这个值，则提示

```sql
ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. You have to change some columns to TEXT or BLOBs。
```

针对第四个问题：若定义的时候超过上述限制，则varchar字段会被强行转为text类型，并产生warning。

##### 3.  与char的对比

CHAR(M)定义的列的长度为固定的，M取值可以为0～255之间，当保存CHAR值时，在它们的右边填充空格以达到指定的长度。当检 索到CHAR值时，尾部的空格被删除掉。在存储或检索过程中不进行大小写转换。CHAR存储定长数据很方便，**CHAR字段上的索引效率级高**，**比如定义 char(10)，那么不论你存储的数据是否达到了10个字节，都要占去10个字节的空间,不足的自动用空格填充。**

CHAR和VARCHAR最大的不同就是一个是固定长度，一个是可变长度。由于是可变长度，因此实际存储的时候是实际字符串再加上一个记录 字符串长度的字节(如果超过255则需要两个字节)。**如果分配给CHAR或VARCHAR列的值超过列的最大长度，则对值进行裁剪以使其适合**。如果被裁掉 的字符不是空格，则会产生一条警告。如果裁剪非空格字符，则会造成错误(而不是警告)并通过使用严格SQL模式禁用值的插入。 **有可能导致数据的不完整性。**

##### 5.  char、varchar与text的建议 

TEXT只能储存纯文本文件。

**效率来说基本是char>varchar>text**，但是如果使用的是Innodb引擎的话，推荐使用varchar代替char。

char和varchar可以有默认值，text不能指定默认值。 

char是固定长度，所以它的处理速度比varchar快得多，但缺点是浪费存储空间，不能在行尾保存空格。

##### 6.  char定长。binary类似于char,binary只能保存二进制字符串。

##### 7.  text与blob区别

blob保存二进制数据；text保存字符数据，有字符集。text和blob不能有默认值。

应用：text与blob主要区别是text用来保存字符数据（如文章，日记等），blob用来保存二进制数据（如照片等）。blob与text在执行了大量删除操作时候，有性能问题（产生大量的“空洞“），为提高性能建议定期optimize table 对这类表进行碎片整理。



# 外键与索引

外键是一种约束，与索引的概念不一样，只是大多数情况下，我们建立外键时，都会在外键列上建立对应的索引。外键的存在会在每一次数据插入、修改时进行约束检查，如果不满足外键约束，则禁止数据的插入或修改，这必然带来一个问题，就是在数据量特别大的情况下，每一次约束检查必然导致性能的下降。索引其实也有类似的问题，**索引如果建多了，那么在插入删除修改数据时也要去维护对应的索引，所以索引的存在也会导致数据操作变慢。**

出于性能的考虑，如果我们的系统完全由我们开发的程序使用，而不需要提供数据库给其他应用系统写入数据，而且对性能要求较高，那么我们**可以考虑在生产环境中不使用外键，只需要建立能够提高性能的索引**。

**感觉这也就是我们开发中， 由业务代码来维护表的关系， 实际上不去建立真正的外键关系。** 



# 分表和分区



## 为什么我们要分表分区？

日常开发中我们经常会遇到大表的情况，所谓的大表是指存储了百万级乃至千万级条记录的表。这样的表过于庞大，导致数据库在查询和插入的时候耗时太长，性能低下，如果涉及联合查询的情况，性能会更加糟糕。分表和表分区的目的就是减少数据库的负担，提高数据库的效率，通常点来讲就是提高表的增删改查效率。

## 分表分区原则

分表主要目的是为突破单节点数据库服务器的 I/O 能力限制，解决数据库扩展性问题。 同时分表分库等思想也将引出以后的数据库集群，主从复制、读写分离方案…

## 分区

分区和分表相似，都是按照规则分解表。不同在于分表将大表分解为若干个独立的实体表，而分区是将数据分段划分在多个位置存放，可以是同一块磁盘也可以在不同的机器。分区后，表面上还是一张表，但数据散列到多个位置了。**app读写的时候操作的还是大表名字，db自动去组织分区的数据。**

## 分表

### 定义

分表是将一个大表按照一定的规则分解成多张具有独立存储空间的实体表，我们可以称为子表，每个表都对应三个文件，MYD数据文件，.MYI索引文件，.frm表结构文件。这些子表可以分布在同一块磁盘上，也可以在不同的机器上。**app读写的时候根据事先定义好的规则得到对应的子表名，然后去操作它。**


### 分表拆分方式

#### 1.  垂直切分

![20170603130517917](/img/20170603130517917.png)

##### 定义

把主键和一些数据表的列放在一个表中，然后把主键和另一些数据表的列放在一个表中。

如果一个表的某些列常用，另一些不常用，则可以采用垂直拆分。垂直拆分可以使数据行变小，一个数据页就可以存放更多的数据，在查询时候可以减少I/O次数。其缺点是需要管理冗余列，查询所有数据时候需要join查找。

##### 优点

使得行数据变小，一个数据块(Block)就能存放更多的数据，在查询时就会减少I/O次数(每次查询时读取的Block 就少)。

##### 缺点

表垂直分割后，主码(主键)出现冗余，需要管理冗余列。

会引起表连接JOIN操作（增加CPU开销）需要从业务上规避。



#### 2.  水平拆分（分表，分区）–按表中某一字段值的范围划分

![20170603130556569](/img/20170603130556569.png)

##### 定义

根据列的范围值进行合理切分，放在多个独立的表或分区中。

##### 适用场景

1. 表很大，分割后可以降低查询时候需要读取的数据和索引的页数，同时降低索引的层数，提高查询速度。

2. 表中的数据是独立的，例如表中分别记录各个地区的数据或不同时期的数据，特别是有些数据常用，而另一些数据不常用。
3. 需要把数据放在多个存储介质上。
4. 需要把历史数据和当前的数据拆分开。

##### 例子

当伴随着某一个表的数据量越来越大，以至于不能承受的时候，就需要对它进行进一步的切分。一种选择是根据key 的范围来做切分，譬如ID 为 1-10000的放到表A上，ID 为10000~20000的放到表B。这样的扩展就是可预见的。另一种是根据某一字段值来划分，譬如根据用户名的首字母，如果是A-D，就属于表A，E-H就属于表B。这样做也存在不均衡性，当某个范围超出了单点所能承受的范围就需要继续切分。还有按日期切分等等。

##### 优点

- 不存在单库大数据和高并发的性能瓶颈
- 应用端改造较少
- 提高了系统的稳定性和负载能力

##### 缺点

- 分片事务一致性难以解决
- 跨节点Join性能差，逻辑复杂
- 数据多次扩展难度跟维护量极大

#### 3.   散列库表（基于hash算法的切分）

##### 定义

表散列与水平分割相似，但没有水平分割那样的明显分割界限，采用Hash算法把数据分散到各个分表中, 这样IO更加均衡。一般采用mod来切分，一开始确定切分数据库的个数，通过hash取模来决定使用哪台。这种方法能够平均地来分配数据，但是伴随着数据量的增大，需要进行扩展的时候，这种方式无法做到在线扩容。每增加节点的时候，就需要对hash 算法重新运算。

我们会按照业务或者功能模块将数据库进行分离，不同的模块对应不同的数据库或者表，再按照一定的策略对某个页面或者功能进行更小的数据库散列，比如用户表，按照用户ID进行表散列，散列128张表，则应就能够低成本的提升系统的性能并且有很好的扩展性。

##### 优点

数据分布均匀

##### 缺点

数据迁移的时候麻烦，不能按照机器性能分摊数据



#### 4. Mrg_Myisam引擎实现水平分表

##### 优点

单表大小可控，天然水平扩展。降低在查询时需要读的数据和索引的页数，同时也降低了索引的层数，加快了查询速度。

##### 缺点

无法解决集中写入瓶颈的问题。同时，水平分割会给应用增加复杂度，它通常在查询时需要多个表名，查询所有数据需要union操作。在许多数据库应用中，这种复杂性会超过它带来的优点，因为只要索引关键字不大，则在索引用于查询时，表中增加两到三倍数据量，查询时也就增加读一个索引层的磁盘次数。



### 分区和分表相关问题总结

#### 1.  分表和分区不矛盾，可以相互配合

#### 2.  分表技术是比较麻烦的，需要手动去创建子表，app服务端读写时候需要计算子表名。

#### 3.  对记录多的表进行拆分。（上千万级别的表）

#### 4.  需要拆分的表分为动态表和相对静态表。

动态表拆分到不同库，静态表存在于公共库。从公共库同步到分库。实现表的连接。

#### 5.  按照年、月、地域等来分割，或者根据时间范围

按照年、月、地域等来分割，或者根据时间范围、和很固定又清晰的字段值范围等，具有确定的分割标志来分割。

#### 6.  通过MyCat的来处理水平分表后， 查询多个字表的问题。 

<https://blog.csdn.net/baogeda/article/details/78425813>

# 实际开发中设计思考和优化

### 1.  适当的冗余，增加计算列

数据库设计的实用原则是：在数据冗余和处理速度之间找到合适的平衡点。

满足范式的表一定是规范化的表，但不一定是最佳的设计。很多情况下会为了提高数据库的运行效率，常常需要降低范式标准：适当增加冗余，达到以空间换时间的目的。比如我们有一个表，产品名称，单价，库存量，总价值。这个表是不满足第三范式的，因为“总价值”可以由“单价”乘以“数量”得到，说明“金额”是冗余字段。但是，增加“总价值”这个冗余字段，可以提高查询统计的速度，这就是以空间换时间的作法。合理的冗余可以分散数据量大的表的并发压力，也可以加快特殊查询的速度，冗余字段可以有效减少数据库表的连接，提高效率。

### 2.  索引的设计 

表优化的重要途径，比如百万级别的表没有索引，注定卡死。

### 3.  存储过程、视图、函数的适当使用

1. 存储过程减少了网络传输、处理及存储的工作量，且经过编译和优化，执行速度快，易于维护，且表的结构改变时，不影响客户端的应用程序
2. 使用存储过程，视图，函数有助于减少应用程序中SQL复制的弊端，因为现在只在一个地方集中处理SQL

### 4.  分割你的表，减小表尺寸

如果你发现某个表的记录太多，例如超过一千万条，则要对该表进行水平分割。水平分割的做法是，以该表主键的某个值为界线，将该表的记录水平分割为两个表。

如果你若发现某个表的字段太多，例如超过八十个，则垂直分割该表，将原来的一个表分解为两个表。

### 5.  字段设计原则

#### 1.   数据类型尽量用数字型，数字型的比较比字符型的快很多。

```txt
尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连 接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了；
```

#### 2.   数据类型尽量小，这里的尽量小是指在满足可以预见的未来需求的前提下的。

#### 4.   少用TEXT和IMAGE，二进制字段读写比较慢，读取的方法也不多，大部分情况下最好不用。

#### 5.   自增字段要慎用，不利于数据迁移

#### 6.   尽量使用 tinyint、smallint、mediumint 作为整数类型而非 int

#### 7.   尽可能使用 not null 定义字段，因为 null 占用4字节空间，同时对索引和性能很大影响

```
NULL 类型比较特殊，SQL 难优化。虽然 MySQL NULL类型和 Oracle 的NULL 有差异，会进入索引中，但如果是一个组合索引，那么这个NULL 类型的字段会极大影响整个索引的效率。此外，NULL 在索引中的处理也是特殊的，也会占用额外的存放空间。

尽量避免NULL：应该指定列为NOT NULL，除非你想存储NULL。在MySQL中，含有空值的列很难进行查询优化。因为它们使得索引、索引的统计信息以及比较运算更加复杂。你应该用0、一个特殊的值或者一个空串代替空值。


```

#### 7.   空字符串('')的长度是0，是不占用空间的

```
完全可以用它来设置默认值
```

#### 8.   尽量使用 timestamp 而非 datetime

#### 9.   单表不要有太多字段，建议在 20 以内

#### 10. 尽可能的使用 varchar/nvarchar 代替 char/nchar

```
因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些；
```

#### 11. 尽量不要使用text数据类型 

```
除非使用text处理一个很大的数据，否则不要使用它。因为它不易于查询，速度慢，用的不好还会浪费大量的空间。一般varchar可以更好的处理数据
```

#### 12.  用整型来存IP

#### 13.  如果存储的字符串长度几乎相等，使用 char 定长字符串类型。

#### 14.  合适的字符存储长度，不但节约数据库表空间、节约索引存储，更重要的是提升检 索速度。

#### 15.  业务上具有唯一特性的字段，即使是多个字段的组合，也必须建成唯一索引



# 表命名规则

转载地址：https://zhuanlan.zhihu.com/p/58650926

#### 1.  表达是与否概念的字段，必须使用 is_xxx 的方式命名

#### 2.  表名、字段名必须使用小写字母或数字

#### 3.  主键索引名为 pk_字段名；唯一索引名为 uk_字段名；普通索引名则为 idx_字段名

#### 4.  表的命名最好是加上“业务名称+表的作用，模块名称+具体表名”

#### 5.  如果修改字段含义或对字段表示的状态追加时，需要及时更新字段注释。

#### 6.  多对多的连接表可以使用两个表的前缀作为表名

```sql
如：用户登录表User_Login，用户分组表User_GroupInfo，这两个表建立多对多关系的表名为：User_Group_Relation（关系统一用Relation）

user_r_group  这样也可以,对于数据库快捷键很方便的就出来了。
```

#### 7.  表名称不应该取得太长（一般不超过三个英文单词）

#### 8.  表名的单词一般都为单数

#### 9.  日志表一般以Sys_开头，数据字典表一般以SD_开头，系统字典表一般以DT_开头

#### 10. 常用表名约定

```sql
user 用户
category 分类
goods 商品、物品
good_gallery 物品相册
good_cate 物品分类
attr 属性
article 文章
cart 购物差
feedback 用户反馈
order 订单
site_nav 页头和页尾导航
site_config 系统配置表
admin 后台用户                    sys_admin
role 后台用户角色                  sys_role
access 后台操作权限                sys_access
role_admin 后台用户对应的角色       sys_role_admin       role_r_admin    其他表用 r 表示关系
access_role 后台角色对应的权限      sys_access_role      access_r_role   其他表用 r 表示关系
```

#### 11.  操作日志表，登录日志表，这是数据库中必备的两个表

```sql
这个记录也需要做进一步的保存。这个有两种情形，一是具体到单个字段的操作日志，二是整个表的操作日志。
常见的几个表具体说明：
操作日志表Sys_OperateLog、登录日志表Sys_LoginLog、
系统字典表Sys_Dictionary，系统字典表类型Sys_DicType

```

#### 12.  是别的表的外键均使用xxx_id的方式来表明

#### 13.  char定长的比varchar()更适合做MD5加密的password，他的查询速度更快，没有碎片

#### 14.  所有的临时表必须以tmp_ 开头，备份表必须以bak_ 开头，并以日期为后缀，例如tmp_user_20190721，便于标识表，和后面清理数据作为依据

#### 15.  说明:任何字段如果为非负数，必须是 unsigned。

#### 16.   表必备字段

```sql
create_date`，`last_update_date`，`create_by`， `last_update_by`，`object_version_number`。也可以叫做Who字段，就是每个表里必须具备的字段。这些字段起到似metadata的作用。这些字段的作用很大，例如，数据分析的时候，可以使用`last_update_date`作为数据抽取的时间戳字段等。

- `id` 必为主键，类型为 `unsigned bigint`、单表时自增、步长为 1。
- `create_date` 是此条数据的创建时间，数据类型为`datetime` 类型。
- `last_update_date`是此条数据的最后更新时间，数据类型为 `datetime` 类型。
- `create_by`是此条数据的创建人，数据类型为`unsigned bigint`类型。
- `last_update_by`是此条数据的最后更新人，数据类型为`unsigned bigint`类型。
- `object_version_number`是此条数据的版本号，如果启用数据库数据版本控制，则会使用到此数据。
```

#### 17.  数据库命名以环境结尾（_dev）；

#### 18.  外键以（FK_）开头，全部大写；

#### 19.  如果是后台表命名时应该在表名基础上加上后缀_b



# 数据库设计常见表

转载地址：https://www.linuxidc.com/Linux/2018-09/154035.htm

### 1.字典表(sys_dict)

作用：用于存放多组值不变的基础数据,只对系统提供查询功能.

![180912101394532](/img/180912101394532.png) 

1. 记录的新增、更新、删除都是通过手动进行操作.
2. 其中dict_code为dict_title的编码,相同dict_title的记录为同一组基础数据,每组基础数据下又有多对dict_value与dict_name.
3. 每组基础数据可以根据实际的业务需求在程序中创建对应的枚举类(value和name属性).



### 2.系统配置表(sys_config)

作用：用于存放系统的配置项,某些业务逻辑需要根据配置项的值来做出相应的处理.

![180912101394531-1](/img/180912101394531-1.png)

1. 记录的新增、删除都是通过手动进行操作.
2. 在系统配置页面中查询配置项并修改配置项的值.
3. 在某些业务逻辑中需根据模块ID和配置代码查询配置项,根据不同的配置值做出相应的处理. 
4. 可以创建一个枚举类存放模块ID,创建常量类存放config_code.



### 3.地域表(sys_area)

作用：用于存放省市区地域数据,一般只对系统提供查询功能.

![180912101394539](/img/180912101394539.png)

1. 记录的新增、更新、删除都是通过手动进行操作. 

2. 在页面中通过多级联动选择地域,调用根据父编码查询记录的API(首次查询父编码为0的记录表示顶层节点)



### 4.RBAC

#### 用户表(sys_user)

![180912101394533](/img/180912101394533.png)



#### 角色表(sys_role)

![180912101394538](/img/180912101394538.png)



#### 菜单表(sys_menu)

![180912101394534](/img/180912101394534.png)



#### 用户角色关联表(sys_user_role)

![180912101394535](/img/180912101394535.png)

1. 其中user_id和role_id为联合主键,可以保证一个用户不会存在相同的角色.



#### 角色菜单关联表(sys_role_menu)

![180912101394536](/img/180912101394536.png)

1. 其中role_id和menu_id为联合主键,可以保证一个角色不会存在相同的权限.



### 5. 机构表(sys_office)

作用：用于存放公司的组织架构关系(适用于集团)

![180912101394537](/img/180912101394537.png)



### 6. 系统操作日志(sys_log)

作用：用于记录用户在系统中的操作行为. 

1. 系统操作日志功能一般会进行日志的输出以及数据的入库.
2. 系统操作日志表由于数据量众多,因此需要在查询参数中添加索引. 

![1809121013945311](/img/1809121013945311.png)



![1809121013945310](/img/1809121013945310.png)


---
title:       "mongodb-入门"
subtitle:    ""
description: ""
date:        2019-09-15
author:      "麦子"
image:       "https://c.pxhere.com/images/a1/23/147c2090573fcd3fbc38e7b7b5a7-1593571.jpg!d"
tags:        ["mongodb", "$type运用", "capped collection", "objectid分析"]
categories:  ["Tech" ]
---

[TOC]

**说明：本文来自于<https://www.bilibili.com/video/av36394924/?p=2>视频讲解**

**官方文档：<https://docs.mongodb.com/>  4.2版本**

**中文社区：[http://mongoing.com/**](http://mongoing.com/)

# mongodb优势

它是一个内存数据库，数据都是放在内存里面的。对数据的操作大部分都在内存中，但mongodb并不是单纯的内存数据库。

查询性能是MongoDB的强项之一。它将大部分可工作的数据存储在RAM中。所有数据都保留在硬盘中，但在查询期间，它不会从硬盘中获取数据。它相当于从本地RAM获取，因此能够提供更快的速度。在这里，重要的是要有正确的索引和足够大的RAM来从MongoDB的性能中获益。

# mongodb数据丢失否

转载地址：https://blog.csdn.net/yibing548/article/details/50844310

MongoDB会先把数据更新写入到Journal Buffer里面然后再更新内存数据，然后再返回给应用端。Journal会以100ms的间隔批量刷到盘上。这样的情况下，即使出现断电数据尚未保存到文件，由于有Journal文件的存在，MongoDB会自动根据Journal里面的操作历史记录来对数据文件重新进行追加。

Journal文件是100ms 刷盘一次。那么要是系统掉电正好发生在上一次刷journal的50ms之后呢？这个时候，我们就可以来看一下MongoDB持久化的下一个概念了：写关注
写关注(Write Concern)
写关注（或翻译为写安全机制）是MongoDB特有的一个功能。它可以让你灵活地指定你写操作的持久化设定。这是一个在性能和可靠性之间的一个权衡。

# MongoDB和MySQL选择

例如，许多电子商务应用程序使用MongoDB和MySQL的组合。产品目录包括具有不同属性的多个产品，非常适合MongoDB的灵活数据模型。另一方面，需要复杂事务的结帐系统可能建立在MySQL或其他关系数据库技术上。

# 什么是仲裁节点

经过大量血的教训，一个分片配置两个副本集时（一个是primary一个是secondary），如果primary挂掉，secondary是不会升级的，必须要加上一个不存储数据的仲裁节点

# 读写锁

当进行读操作的时候会加读锁，这个时候其他读操作可以也获得读锁。但是不能或者写锁。

当进行写操作的时候会加写锁，这个时候不能进行其他的读操作和写操作。

所以按照这个道理，是不会出现同时修改同一个文档（如执行++操作）导致数据出错的情况。

而且按照这个道理，因为写操作会阻塞读操作，所以是不会出现脏读的。

但是mongodb在分片和复制集的时候会产生脏读，后面在研究。

**在3.0之后的版本，WiredTiger提供了文档（不是集合）级别的锁。**



# mogodb VS mysql

mogodb是无模式的文档数据库。 

| mysql    | mogodb     | 说明          |
| -------- | ---------- | ------------- |
| database | database   | 数据库/数据库 |
| table    | collection | 表/集合       |
| row      | document   | 行/文档       |



## 区别

**row**

每一行都是一样的数据， 不可以添加不可以减少，也就是说fileds的个数在定义table的时候就一定申明好了。 

**document**

他的每一个document都是独立的，同时也不是我们在create.collection的时候申明好的。 



# document

## 长度大小限制

mogodb天生就是分布式的，可扩展，高性能，遵循CAP。**既然是分布式部署，存在两个问题，内存->粒度太多  网络带宽-> 多而碎,  他的长度是16M。redis的value是512M(String)**



# capped collection(上线集合)

**转载地址：<https://www.jianshu.com/p/86942c09094a>**



## 简介

1. Capped集合是一个固定大小，高性能的，文档按照插入顺序的一个集合。
   新的对象会把覆盖旧的对象，像环形缓存一样。
2. find时默认就是插入的顺序,Capped集合会自动维护。
3. Capped 集合用来解决top 多少的问题，最新的top条评论，最活跃的top用户...
4. 一些日志的话， 或者是发送一些短信消息数量太大过多，存放时间不会太长，使用这个老的替换新的。 自动维护

指定这个collection的大小和document的数目。 

```shell
Maximum size in bytes:  大小
Maximum number of documents: 数据
```

他相当于一个环形的队列， 新的document进来的时候，会替换掉老的document文档。 



## 创建

```sql
db.createCollection("集合名称", { capped : true, size : num, max : num } )
db.createCollection("log", { capped : true, size : 1000, max : 5 } )
```

1. size用来指定集合大小，单位KB
2. 限制集合中对象的个数：可以在创建时设置max参数
3. 指定mac数量的时候必须同时指定size容量。淘汰机制只有在**容量还没有满时才会依据文档数量工作**。要是容量满了，淘汰机制会依据容量来工作。



## 列子

设置最大是3个document：

```sql
db.getCollection("testcapped").insert({name:1})
db.getCollection("testcapped").insert({name:2})
db.getCollection("testcapped").insert({name:3})
db.getCollection("testcapped").insert({name:4})
```

然后查看这个collection

```json
db.getCollection("testcapped").find();

{ 
    "_id" : ObjectId("5d7de039eae3e224578750ee"), 
    "name" : 2.0
}
// ----------------------------------------------
{ 
    "_id" : ObjectId("5d7de039eae3e224578750ef"), 
    "name" : 3.0
}
// ----------------------------------------------
{ 
    "_id" : ObjectId("5d7de039eae3e224578750f0"), 
    "name" : 4.0
}
```

可以看到最早插入的document已经被覆盖了。 

## API命令操作

```sql
#排序 
db.cappedCollection.find().sort( { $natural: -1 } ) 

#是否是上线集合
db.collection.isCapped()

#把普通集合转为上线集合
db.runCommand({"convertToCapped": "mycoll", size: 100000, max: 100});

```



# bson

[BSON](https://docs.mongodb.com/manual/reference/glossary/#term-bson)是一种二进制序列化格式，用于存储文档并在MongoDB中进行远程过程调用。



## 支持的数据类型

| Type                    | Number | Alias                 | Notes               |
| :---------------------- | :----- | :-------------------- | :------------------ |
| Double                  | 1      | “double”              |                     |
| String                  | 2      | “string”              |                     |
| Object                  | 3      | “object”              |                     |
| Array                   | 4      | “array”               |                     |
| Binary data             | 5      | “binData”             |                     |
| Undefined               | 6      | “undefined”           | Deprecated.         |
| ObjectId                | 7      | “objectId”            |                     |
| Boolean                 | 8      | “bool”                |                     |
| Date                    | 9      | “date”                |                     |
| Null                    | 10     | “null”                |                     |
| Regular Expression      | 11     | “regex”               |                     |
| DBPointer               | 12     | “dbPointer”           | Deprecated.         |
| JavaScript              | 13     | “javascript”          |                     |
| Symbol                  | 14     | “symbol”              | Deprecated.         |
| JavaScript (with scope) | 15     | “javascriptWithScope” |                     |
| 32-bit integer          | 16     | “int”                 |                     |
| Timestamp               | 17     | “timestamp”           |                     |
| 64-bit integer          | 18     | “long”                |                     |
| Decimal128              | 19     | “decimal”             | New in version 3.4. |
| Min key                 | -1     | “minKey”              |                     |
| Max key                 | 127    | “maxKey”              |                     |



## $type运用

```sql
#查询password的类型是字符串的数据
db.getCollection("teacher").find({"password":{$type:2}})
```

**注意：还可以通过正则表达式来处理数据。**



# ObjectId

默认是document的主键， 默认索引。 

12字节的[ObjectId](https://docs.mongodb.com/manual/reference/bson-types/#objectid) 值包括：

- 一个4字节的值，表示自Unix纪元以来的秒数，
- 一个5字节的随机值，和
- 一个3字节的计数器，以随机值开始。

可以获取当前创建这个数据的时间

```javascript
var createTime  = ObjectId("5d7de7d74374945f03b8b592").getTimestamp();
var year = createTime.getFullYear();  // 获取完整的年份(4位,1970)
var month = createTime.getMonth() + 1;  // 获取月份(0-11,0代表1月,用的时候记得加上1)
var date = createTime.getDate();  // 获取日(1-31)
var time = createTime.getTime();  // 获取时间(从1970.1.1开始的毫秒数)
var hours = createTime.getHours();  // 获取小时数(0-23)
var minutes = createTime.getMinutes();  // 获取分钟数(0-59)
var seconds = createTime.getSeconds();  // 获取秒数(0-59)
print(year+" "+month+" "+date+" "+time+" "+hours+" "+minutes+" "+seconds);
```



# shell

mongodb是封装了一个V8引擎，**可以运行js代码的。**可以通过load("js路径")执行里面的mongdb的命令操作。

```shell
#shell启动位置
/bin/mongo
```

## js写法

```javascript
var mongo = new Mongo("localhost:27017");
var db = mongo.getDB("mydatabase"); 
var collection = db.getCollection("student");
var list = collection.find({}).toArray();
printjson(list);
```




---
title:       "mongodb-aggregation pipeline聚合"
subtitle:    ""
description: ""
date:        2019-09-16
author:      "麦子"
image:       "https://c.pxhere.com/images/a1/23/147c2090573fcd3fbc38e7b7b5a7-1593571.jpg!d"
tags:        ["mongodb","聚合操作","管道操作"]
categories:  ["Tech" ]
---

[TOC]

**说明：本文来自于<https://www.bilibili.com/video/av36394924/?p=2>视频讲解**

**官方文档：<https://docs.mongodb.com/manual/reference/operator/aggregation/sample/>  4.2版本**

# aggregation pipeline 简介



## 管道模型

后面操作的数据源来自前面的一个操作



## 用来做什么

 做聚合操作, 分组操作， 但是更加的强大。 **可以跨机器查询。** 



# 常用聚合管道

```sql
规范：
db.collection.aggregate( [ { <stage> }, ... ] )

db.orders.explain().aggregate(
  [
   { $match: { status: "A" } },
   { $group: { _id: "$cust_id", total: { $sum: "$amount" } } },
   { $sort: { total: -1 } }
  ],
  {
    #可选的。允许写入临时文件。设置为时true，聚合操作可以将数据写入_tmp目录中的 dbPath子目录。
    allowDiskUse: true 
   }
)
```



## $project

相当于我们sql中的select， 确定你需要返回哪几个字段

```sql
规范：
{ $project: { <specification(s)> } }

db.books.aggregate( [ { $project : { title : 1 , author : 1 } } ] )
```





## $match

相当于我们sql中的where

```sql
规范：
{ $match: { <query> } }
```



## $limit

相当于我们sql中的limit

```sql
规范：
{ $limit: <positive integer> }

db.article.aggregate(
    { $limit : 5 }
);
```



## $skip

也相当于我们sql中的limit

## $unwind

将数据拆解成每一个document

## $group

这个是分组计算的

```sql
规范：
{ $group: { _id: <expression>, <field1>: { <accumulator1> : <expression1> }, ... } }

db.books.aggregate(
   [
     { $group : { _id : "$author", books: { $sum: "$price" } } }
   ]
)

author这个值用来分组， books这个值就是新的字段
```

### 注意

该[`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group)阶段的RAM限制为100兆字节。默认情况下，如果阶段超出此限制，[`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group)将产生错误。但是，要允许处理大型数据集，请将[`allowDiskUse`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#db.collection.aggregate)选项设置 `true`为启用[`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group)操作以写入临时文件。



## $ample

随机选择文档

## $sort

排序

## $lookup

```sql
规范：
{
   $lookup:
     {
       from: <collection to join>, 
       localField: <field from the input documents>,
       foreignField: <field from the documents of the "from" collection>,
       as: <output array field>
     }
}

#相当于一个左关联
db.orders.aggregate([
   {
     $lookup:
       {
         from: "inventory",
         localField: "item",
         foreignField: "sku",
         as: "inventory_docs"
       }
  }
])

SELECT *, inventory_docs  FROM orders
                          WHERE inventory_docs IN
                                (SELECT * FROM inventory WHERE sku= orders.item);
```

sql中的join操作， 表关联

## $out

将最后的结果指定到collection中， 因为前面的查询都是临时结果， 没有序列化处理。 

```sql
规范：
{ $out: "<output-collection>" }

db.books.aggregate( [
                      { $group : { _id : "$author", books: { $push: "$title" } } },
                      { $out : "authors" }
                  ] )
这样就把books的结果集序列化到了 authors的collection中了。
```



## $indexStats

查询过程中的索引情况

## $redact

决定内嵌文档是否keep保留


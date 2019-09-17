---
title:       "mongodb-crud操作"
subtitle:    ""
description: ""
date:        2019-09-15
author:      "麦子"
image:       "https://c.pxhere.com/images/a1/23/147c2090573fcd3fbc38e7b7b5a7-1593571.jpg!d"
tags:        ["mongodb","常用crud操作"]
categories:  ["Tech" ]
---

[TOC]

**说明：本文来自于<https://www.bilibili.com/video/av36394924/?p=2>视频讲解**

**官方文档：<https://docs.mongodb.com/>  4.2版本**

# insert

## 通用

```sql
规范： 
db.collection.insert(
   <document or array of documents>,
   {
     writeConcern: <document>,
     ordered: <boolean>
   }
)


```

**ordered**

如果`true`，在数组中执行文档的有序插入，并且如果其中一个文档发生错误，MongoDB将返回而不处理数组中的其余文档。如果`false`执行无序插入，并且如果其中一个文档发生错误，请继续处理阵列中的其余文档。**默认为true**



## insertOne

```sql
规范：
db.collection.insertOne(
   <document>,
   {
      writeConcern: <document>
   }
)

db.inventory.insertOne(
   { item: "canvas", qty: 100, size: { h: 28, w: 35.5, uom: "cm" } }
)
```



## insertMany

```sql
规范：
db.collection.insertMany(
   [ <document 1> , <document 2>, ... ],
   {
      writeConcern: <document>,
      ordered: <boolean>
   }
)

db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])
```



# find



## 通用

```sql
{ field1: <value>, field2: <value> ... }
```



## findOne

```sql
规范：
{ field1: <boolean>, field2: <boolean> ... }

db.getCollection("inventory").findOne({},{qty:1,tags:1})

其中 1 表示，这个字段会输出出来
其中 0 表示， 这个字段不会输出出来
```



## findAndModify

```sql
规范：
db.collection.findAndModify({
    query: <document>,
    sort: <document>,
    remove: <boolean>,
    update: <document or aggregation pipeline>, // Changed in MongoDB 4.2
    new: <boolean>,
    fields: <document>,
    upsert: <boolean>,
    bypassDocumentValidation: <boolean>,
    writeConcern: <document>,
    collation: <document>,
    arrayFilters: [ <filterdocument1>, ... ]
});

db.getCollection("inventory").findAndModify({
    query: { qty: 15 },
    sort: { qty: 1 },
    update: { $inc: { qty: 100 } },
    upsert: true
})  
```



## findOneAndDelete

```sql
规范：
db.collection.findOneAndDelete(
   <filter>,
   {
     projection: <document>, #可选的。要返回的字段子集。
     sort: <document>,
     maxTimeMS: <number>, #指定操作必须在其中完成的时间限制（以毫秒为单位）
     collation: <document> #指定 要用于操作的排序规则
   }
)


db.getCollection("inventory").findOneAndDelete(
   { "item.name" : "ab" }
)
```



## findOneAndReplace()

```sql
规范：
db.collection.findOneAndReplace(
   <filter>,             #{ field1 : < boolean >, field2 : < boolean> ... }
   <replacement>,        # 替换文件，不能包含 更新运算符。
   {
     projection: <document>,
     sort: <document>,
     maxTimeMS: <number>,
     upsert: <boolean>,
     returnNewDocument: <boolean>,
     collation: <document> #指定 要用于操作的排序规则
   }
)

db.scores.findOneAndReplace(
   { "score" : { $lt : 20000 } },
   { "team" : "Observant Badgers", "score" : 20000 }
)
```



## findOneAndUpdate

```sql
规范：
db.collection.findOneAndUpdate(
   <filter>,
   <update document or aggregation pipeline>, // Changed in MongoDB 4.2
   {
     projection: <document>,
     sort: <document>,
     maxTimeMS: <number>,
     upsert: <boolean>,
     returnNewDocument: <boolean>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)

db.grades.findOneAndUpdate(
   { "name" : "R. Stiles" },
   { $inc: { "points" : 5 } }
)
```

# delete

## deleteOne

```sql
规范：
db.collection.deleteOne(
   <filter>,
   {
      writeConcern: <document>,
      collation: <document>
   }
)

 db.orders.deleteOne( { "expiryts" : { $lt: ISODate("2015-11-01T12:40:15Z") } } );
```



## deleteMany

```sql
规范：
db.collection.deleteMany(
   <filter>,
   {
      writeConcern: <document>,
      collation: <document>
   }
)

 db.orders.deleteMany( { "client" : "Crude Traders Inc." } );
```



## remove

```sql
规范：
db.collection.remove(
   <query>,
   <justOne>  #要将删除限制为仅一个文档，请设置为true。省略使用默认值false并删除符合删除条件的所有文档。
)

db.products.remove( { qty: { $gt: 20 } } )
```



# update

```sql
规范：
db.collection.update(
   <query>,
   <update>,
   {
     #如果设置为true，则在没有文档与查询条件匹配时创建新文档。默认值为false，未找到匹配项时不插入新文档。
     upsert: <boolean>, 
     multi: <boolean>, 
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ],
     hint:  <document|string>        // Available starting in MongoDB 4.2
   }
)

db.people.update(
   { name: "Andy" },
   {
      name: "Andy",
      rating: 1,
      score: 1
   },
   { upsert: true }
)
```

**注意：默认更新单个文档。包括选项multi：true 以更新符合查询条件的所有文档。**

## updateOne

```sql
规范：
db.collection.updateOne(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)

db.restaurant.updateOne(
      { "name" : "Central Perk Cafe" },
      { $set: { "violations" : 3 } }
   );
```



## updateMany

```sql
规范：
db.collection.updateMany(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)

db.restaurant.updateMany(
      { violations: { $gt: 4 } },
      { $set: { "Review" : true } }
   );
```



# 批量操作

文档提供了很多的批量的方法操作。

**官方地址：<https://docs.mongodb.com/manual/reference/method/js-bulk/>**

```sql
var bulk = db.items.initializeUnorderedBulkOp();
#items`并添加一系列插入操作以添加多个文档
bulk.insert( { item: "abc123", defaultQty: 100, status: "A", points: 100 } );
bulk.insert( { item: "ijk123", defaultQty: 200, status: "A", points: 200 } );
bulk.insert( { item: "mop123", defaultQty: 0, status: "P", points: 0 } );
bulk.execute();
```








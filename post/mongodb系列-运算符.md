---
title:       "mongodb系列-运算符"
subtitle:    ""
description: ""
date:        2019-09-15
author:      "麦子"
image:       "https://c.pxhere.com/images/a1/23/147c2090573fcd3fbc38e7b7b5a7-1593571.jpg!d"
tags:        ["mongodb系列", "比较运算符", "逻辑运算符", "评估查询运算符", "数组查询运算符", "字段更新运算符", "数组更新运算符"]
categories:  ["Tech" ]
---

[TOC]

**说明：本文来自于<https://www.bilibili.com/video/av36394924/?p=2>视频讲解**

**官方文档：<https://docs.mongodb.com/>  4.2版本**

# 比较运算符

|                             名称                             |            描述            |
| :----------------------------------------------------------: | :------------------------: |
| [`$gt`](https://docs.mongodb.com/manual/reference/operator/query/gt/#op._S_gt) |    匹配大于指定值的值。    |
| [`$eq`](https://docs.mongodb.com/manual/reference/operator/query/eq/#op._S_eq) |    匹配等于指定值的值。    |
| [`$gte`](https://docs.mongodb.com/manual/reference/operator/query/gte/#op._S_gte) | 匹配大于或等于指定值的值。 |
| [`$in`](https://docs.mongodb.com/manual/reference/operator/query/in/#op._S_in) |  匹配数组中指定的任何值。  |
| [`$lt`](https://docs.mongodb.com/manual/reference/operator/query/lt/#op._S_lt) |    匹配小于指定值的值。    |
| [`$lte`](https://docs.mongodb.com/manual/reference/operator/query/lte/#op._S_lte) | 匹配小于或等于指定值的值。 |
| [`$ne`](https://docs.mongodb.com/manual/reference/operator/query/ne/#op._S_ne) | 匹配所有不等于指定值的值。 |
| [`$nin`](https://docs.mongodb.com/manual/reference/operator/query/nin/#op._S_nin) | 不匹配数组中指定的任何值。 |

## 

## $in

```sql
格式: { field: { $in: [<value1>, <value2>, ... <valueN> ] } }

db.inventory.find( { qty: { $in: [ 5, 15 ] } } )
```



## $nin

```sql
格式: { field: { $nin: [ <value1>, <value2> ... <valueN> ]} }

db.inventory.find( { qty: { $nin: [ 5, 15 ] } } )
```





## 其他

```sql
格式: {field: {$gt: value} }
db.inventory.find( { qty: { $gt: 20 } } )

格式: {field: {$gte: value} }
db.inventory.find( { qty: { $gte: 20 } } )

格式:  {field: {$lt: value} }
db.inventory.update( { "carrier.fee": { $lt: 20 } }, { $set: { price: 9.99 } } )

格式: { <field>: { $eq: <value> } }
db.inventory.find( { qty: { $eq: 20 } } )
db.inventory.find( { qty: 20 } )

格式:  { field: { $lte: value} }
db.inventory.find( { qty: { $lte: 20 } } )

格式:  {field: {$ne: value} }
db.inventory.find( { qty: { $ne: 20 } } )


```

# 逻辑运算符

|                             名称                             |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| [`$or`](https://docs.mongodb.com/manual/reference/operator/query/or/#op._S_or) | 使用逻辑连接查询子句`OR`将返回与任一子句的条件匹配的所有文档。 |
| [`$and`](https://docs.mongodb.com/manual/reference/operator/query/and/#op._S_and) | 使用逻辑连接查询子句`AND`将返回与两个子句的条件匹配的所有文档。 |
| [`$not`](https://docs.mongodb.com/manual/reference/operator/query/not/#op._S_not) |     反转查询表达式的效果并返回与查询表达式*不*匹配的文档     |
| [`$nor`](https://docs.mongodb.com/manual/reference/operator/query/nor/#op._S_nor) | 使用逻辑连接查询子句`NOR`将返回所有无法匹配两个子句的文档。  |

## $or

```sql
格式: { $or: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] }

db.inventory.find( { $or: [ { quantity: { $lt: 20 } }, { price: 10 } ] } )
```



## $and

```sql
格式: { $and: [ { <expression1> }, { <expression2> } , ... , { <expressionN> } ] }
db.inventory.find( { $and: [ { price: { $ne: 1.99 } }, { price: { $exists: true } } ] } )
```



## $not

```sql
格式:  { field: { $not: { <operator-expression> } } }
db.inventory.find( { price: { $not: { $gt: 1.99 } } } )
```



## $nor

```sql
格式:  { $nor: [ { <expression1> }, { <expression2> }, ...  { <expressionN> } ] }
db.inventory.find( { $nor: [ { price: 1.99 }, { sale: true } ]  } )
```



# 评估查询运算符

|                             名称                             |                       描述                       |
| :----------------------------------------------------------: | :----------------------------------------------: |
| [`$mod`](https://docs.mongodb.com/manual/reference/operator/query/mod/#op._S_mod) | 对字段的值执行模运算，并选择具有指定结果的文档。 |
| [`$regex`](https://docs.mongodb.com/manual/reference/operator/query/regex/#op._S_regex) |        选择值与指定正则表达式匹配的文档。        |
| [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#op._S_text) |                  执行文本搜索。                  |
| [`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#op._S_where) |         匹配满足JavaScript表达式的文档。         |



## $mod

```sql
格式:   { field: { $mod: [ divisor, remainder ] } }

#取模， 模4等于0的
db.inventory.find( { qty: { $mod: [ 4, 0 ] } } )
```



## $regex

性能不高， 全表扫描

```sql
格式:    
{ <field>: { $regex: /pattern/, $options: '<options>' } }
{ <field>: { $regex: 'pattern', $options: '<options>' } }
{ <field>: { $regex: /pattern/<options> } }  这种是常用的

{ name: { $regex: '(?i)a(?-i)cme' } }
```



## $text

MongoDB 从3.2 版本以后添加了对中文索引的支持

```sql
格式：
{
  $text:
    {
      $search: <string>,
      $language: <string>,
      $caseSensitive: <boolean>,
      $diacriticSensitive: <boolean>
    }
}

db.articles.find( { $text: { $search: "coffee" } } )
```



##  $where

全表扫描。 相当于循环每一行的数据， 其中this也就是表示是当前的一行，后面接的是一个function模式。 

```
db.players.find( { $where: function() {
   return (hex_md5(this.name) == "9b53e667f30cd329dca1ec9e6a83e994")
} } );
```



# 数组查询运算符

|                             名称                             |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| [`$all`](https://docs.mongodb.com/manual/reference/operator/query/all/#op._S_all) |             匹配包含查询中指定的所有元素的数组。             |
| [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#op._S_elemMatch) | 如果数组字段中的元素与所有指定[`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#op._S_elemMatch)条件匹配，则选择文档。 |
| [`$size`](https://docs.mongodb.com/manual/reference/operator/query/size/#op._S_size) |             如果数组字段是指定大小，则选择文档。             |



## $all

```sql
格式： { <field>: { $all: [ <value1> , <value2> ... ] } }
{ tags: { $all: [ "ssl" , "security" ] } }

相当于

{ $and: [ { tags: "ssl" }, { tags: "security" } ] }
```



## $elemMatch

只要有一个条件满足，就返回。 针对的事数组

```sql
格式： { <field>: { $elemMatch: { <query1>, <query2>, ... } } }

db.scores.find(
   { results: { $elemMatch: { $gte: 80, $lt: 85 } } }
)
```



## $size

```sql
格式： db.collection.find( { field: { $size: 2 } } );

db.collection.find( { field: { $size: 1 } } );
```



# 字段更新运算符

|                             名称                             |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| [`$currentDate`](https://docs.mongodb.com/manual/reference/operator/update/currentDate/#up._S_currentDate) |      将字段的值设置为当前日期，可以是Date或Timestamp。       |
| [`$inc`](https://docs.mongodb.com/manual/reference/operator/update/inc/#up._S_inc) |                  按指定的数量增加字段的值。                  |
| [`$min`](https://docs.mongodb.com/manual/reference/operator/update/min/#up._S_min) |           仅当指定的值小于现有字段值时才更新字段。           |
| [`$max`](https://docs.mongodb.com/manual/reference/operator/update/max/#up._S_max) |           仅当指定的值大于现有字段值时才更新字段。           |
| [`$mul`](https://docs.mongodb.com/manual/reference/operator/update/mul/#up._S_mul) |                   将字段的值乘以指定的量。                   |
| [`$rename`](https://docs.mongodb.com/manual/reference/operator/update/rename/#up._S_rename) |                         重命名字段。                         |
| [`$set`](https://docs.mongodb.com/manual/reference/operator/update/set/#up._S_set) |                     设置文档中字段的值。                     |
| [`$setOnInsert`](https://docs.mongodb.com/manual/reference/operator/update/setOnInsert/#up._S_setOnInsert) | 如果更新导致文档插入，则设置字段的值。对修改现有文档的更新操作没有影响。 |
| [`$unset`](https://docs.mongodb.com/manual/reference/operator/update/unset/#up._S_unset) |                   从文档中删除指定的字段。                   |



## $inc

```sql
格式：  { $inc: { <field1>: <amount1>, <field2>: <amount2>, ... } }


db.products.update(
   { sku: "abc123" },
   { $inc: { quantity: -2, "metrics.orders": 1 } }
)
把quantity的值减少2， metrics.orders的值加一
```



## $mul

```sql
格式：  { $mul: { <field1>: <number1>, ... } }

db.products.update(
   { _id: 2 },
   { $mul: { price: NumberLong(100) } }
)
price 乘以100
```



## $rename

```sql
格式：  {$rename: { <field1>: <newName1>, <field2>: <newName2>, ... } }

db.students.update( { _id: 1 }, { $rename: { 'nickname': 'alias', 'cell': 'mobile' } } )
```



## $setOnInsert

如果查询不到，就插入一条数据， 这个时候的修改和插入是一个原子性。 

```
格式：  
db.collection.update(
   <query>,
   { $setOnInsert: { <field1>: <value1>, ... } },
   { upsert: true }
)


```



## $set

```sql
格式：  { $set: { <field1>: <value1>, ... } }

db.products.update(
   { _id: 100 },
   { $set: { "details.make": "zzz" } }

  
  #数组
  db.products.update(
   { _id: 100 },
   { $set:
      {
        "tags.1": "rain gear",
        "ratings.0.rating": 2
      }
   }
)
```



## $unset

```sql
格式：  { $unset: { <field1>: "", ... } }

db.products.update(
   { sku: "unknown" },
   { $unset: { quantity: "", instock: "" } }
)
```



## $min

```sql
格式：  { $min: { <field1>: <value1>, ... } }

db.scores.update( { _id: 1 }, { $min: { lowScore: 150 } } )

如果lowScore这个值小于150，就把这个值改成150
```



## $max

```sql
格式：  { $max: { <field1>: <value1>, ... } }

db.scores.update( { _id: 1 }, { $max: { highScore: 950 } } )
```



## $currentDate

```sql
格式：   { $currentDate: { <field1>: <typeSpecification1>, ... } }


db.users.update(
   { _id: 1 },
   {
     $currentDate: {
        lastModified: true,
        "cancellation.date": { $type: "timestamp" }
     }
   }
)
```



# 数组更新运算符

|                             名字                             |                             说明                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| [`$`](https://docs.mongodb.com/manual/reference/operator/update/positional/#up._S_) |         充当占位符以更新与查询条件匹配的第一个元素。         |
| [`$addToSet`](https://docs.mongodb.com/manual/reference/operator/update/addToSet/#up._S_addToSet) |        仅当数组中尚不存在元素时才将元素添加到数组中。        |
| [`$pop`](https://docs.mongodb.com/manual/reference/operator/update/pop/#up._S_pop) |               删除数组的第一个或最后一个项目。               |
| [`$pull`](https://docs.mongodb.com/manual/reference/operator/update/pull/#up._S_pull) |              删除与指定查询匹配的所有数组元素。              |
| [`$push`](https://docs.mongodb.com/manual/reference/operator/update/push/#up._S_push) |                       将项添加到数组。                       |
| [`$pullAll`](https://docs.mongodb.com/manual/reference/operator/update/pullAll/#up._S_pullAll) |                  从数组中删除所有匹配的值。                  |
| [`$each`](https://docs.mongodb.com/manual/reference/operator/update/each/#up._S_each) | 修改[`$push`](https://docs.mongodb.com/manual/reference/operator/update/push/#up._S_push)和[`$addToSet`](https://docs.mongodb.com/manual/reference/operator/update/addToSet/#up._S_addToSet)运算符以附加多个项目以进行阵列更新。 |
| [`$position`](https://docs.mongodb.com/manual/reference/operator/update/position/#up._S_position) | 修改[`$push`](https://docs.mongodb.com/manual/reference/operator/update/push/#up._S_push)运算符以指定数组中添加元素的位置。 |
| [`$slice`](https://docs.mongodb.com/manual/reference/operator/update/slice/#up._S_slice) | 修改[`$push`](https://docs.mongodb.com/manual/reference/operator/update/push/#up._S_push)运算符以限制更新数组的大小。 |
| [`$sort`](https://docs.mongodb.com/manual/reference/operator/update/sort/#up._S_sort) | 修改[`$push`](https://docs.mongodb.com/manual/reference/operator/update/push/#up._S_push)运算符以重新排序存储在数组中的文档 |
|                                                              |                                                              |



## $

```sql
格式：   
db.collection.update(
   { <array>: value ... },
   { <update operator>: { "<array>.$" : value } }
)

#修改了grades的值为82了， 其中{ _id: 1, grades: 80 },中必须要包含有修改的grades 
db.students.updateOne(
   { _id: 1, grades: 80 },
   { $set: { "grades.$" : 82 } }
)
```



## $addToSet

```sql
格式：   { $addToSet: { <field1>: <value1>, ... } }

db.foo.update(
   { _id: 1 },
   { $addToSet: { colors: "c" } }
)
```



## $pop

```sql
格式：  { $pop: { <field>: <-1 | 1>, ... } }

 -1: 第一个 
  1: 最后一个
db.students.update( { _id: 1 }, { $pop: { scores: -1 } } )
```



## $pull

```sql
格式：   { $pull: { <field1>: <value|condition>, <field2>: <value|condition>, ... } }

db.stores.update(
    { },
    { $pull: { fruits: { $in: [ "apples", "oranges" ] }, vegetables: "carrots" } },
    { multi: true }
)
```



## $pullAll

```sql
格式：   { $pullAll: { <field1>: [ <value1>, <value2> ... ], ... } }

db.survey.update( { _id: 1 }, { $pullAll: { scores: [ 0, 5 ] } } )
```



## $push

```sql
格式：   { $push: { <field1>: <value1>, ... } }

db.students.update(
   { name: "joe" },
   { $push: { scores: { $each: [ 90, 92, 85 ] } } }
)
```



## $each

常组合使用

```
 db.students.update(
   { name: "joe" },
   { $push: { scores: { $each: [ 90, 92, 85 ] } } }
)
```



## $slice

数据分片，限制了数组的数量， 如果是正数就是从前面开始，如果是负数就是从后面开始。

```
db.students.update(
   { _id: 2 },
   {
     $push: {
       scores: {
         $each: [ 100, 20 ],
         $slice: 3
       }
     }
   }
)
```



## $sort

```sql
db.students.update(
   { _id: 2 },
   { $push: { tests: { $each: [ 40, 60 ], $sort: 1 } } }
)
```



## $position

插入数组中的下标的位置

```sql
db.students.update(
   { _id: 1 },
   {
     $push: {
        scores: {
           $each: [ 20, 30 ],
           $position: 2
        }
     }
   }
)
```






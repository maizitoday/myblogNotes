---
title:       "mongodb-索引"
subtitle:    ""
description: ""
date:        2019-09-15
author:      "麦子"
image:       "https://c.pxhere.com/images/a1/23/147c2090573fcd3fbc38e7b7b5a7-1593571.jpg!d"
tags:        ["mongodb","常用索引","索引优化","query执行计划"]
categories:  ["Tech" ]
---

[TOC]

**说明：本文来自于<https://www.bilibili.com/video/av36394924/?p=2>视频讲解**

**官方文档： <https://docs.mongodb.com/manual/core/index-single/>  4.2版本**



# 常用索引



## 单键索引

```sql
实例：
db.records.createIndex( { score: 1 } )
db.getCollection("inventory").createIndex({"item.name" : 1})
```



## 复合索引

```sql
规范：
db.collection.createIndex( { <field1>: <type>, <field2>: <type2>, ... } )

db.records.createIndex( { "score": 1 } , {"name" : 1})
```

**注意索引位置：**



## 多键值索引

其实所谓的多键值索引就是在array上面建立一个索引，mongodb会将array的每一个数字进行排序索引。

```sql
规范：
db.coll.createIndex( { <field>: < 1 or -1 > } )

{ _id: 5, type: "food", item: "aaa", ratings: [ 5, 8, 9 ] }
db.inventory.createIndex( { ratings: 1 } )
```



## hash索引

将字段的值通过hash函数变成一个int类型的一个数字，然后放到数组中。 **他是支持分片的。**， 他就是为了定值查找。

```sql
规范：
db.collection.createIndex( { <field>: "hashed" } )
```



## 局部索引

如果你建立的事一个basic indexs， 你解索的索引是针对所有的文档。 既然有所有， 那么你就有相对的词，局部。

他主要是针对你匹配的复合条件的索引，是document的子集。 他有更低的开销，更少的存储空间。 

```sql
规范：
db.restaurants.createIndex(
   { cuisine: 1 },
   { partialFilterExpression: { rating: { $gt: 5 } } }
)
可以使用这几个： $exists $gt, $gte, $lt, $lte  $type  $and  


db.restaurants.find( { cuisine: "Italian", rating: { $lt: 8 } } )
```



## 稀疏索引

每一个文档的字段都不是一样的， 如果这个时候设置一个字段name是其他文档没有这个字段的时候，怎么来建立索引。 

稀疏索引，他的意思就是我建立的index字段必须在有这个字段的document才可以建立。 

```sql
规范：
db.addresses.createIndex( { "xmpp_id": 1 }, { sparse: true } )

db.scores.find( { score: { $lt: 90 } } )
```





## TTL索引

他是一个单键索引， 在制定的时间内，就回消除document数据， 可以用于缓存处理。 

```sql
规范：
db.eventlog.createIndex( { "lastModifiedDate": 1 }, { expireAfterSeconds: 3600 } ) // 单位是秒


```

**注意：** 有延迟， 本身有后台的进程去删除这些document，他是60秒执行一次。 



# 索引管理



## 查看索引

```sql
db.collection.getIndexes()
```



## 移除索引 

```sql
db.pets.dropIndex( "catIdx" )  删除的是字段
db.collection.dropIndexes()  // 会发现_id这个主键索引是删不掉的， 除非换了这个_id
```



## 修改索引

重建操作， 数据量大的时候，磁盘不够了， 碎片整理的，用这个， 但是不建议使用。， 

```
db.collection.reIndex()  
```

这个操作， 只能影响一个mongdb，对主从和分片的其他的mongodb是不会有影响的。 

# 索引的优化

他也是安装BTree的模式来的。

## explain执行计划

```json
db.getCollection("inventory").find({ "item.name" : "cd"}).explain("executionStats")

{ 
    "queryPlanner" : {
        "plannerVersion" : 1.0, 
        "namespace" : "mydatabse.inventory", 
        "indexFilterSet" : false, 
        "parsedQuery" : {
            "item.name" : {
                "$eq" : "cd"
            }
        }, 
        "queryHash" : "9CC5AAF7", 
        "planCacheKey" : "E5AE0C9D", 
        "winningPlan" : {
            "stage" : "FETCH", 
            "inputStage" : {
                "stage" : "IXSCAN", 
                "keyPattern" : {
                    "item.name" : 1.0
                }, 
                "indexName" : "item.name_1", 
                "isMultiKey" : false, 
                "multiKeyPaths" : {
                    "item.name" : [

                    ]
                }, 
                "isUnique" : false, 
                "isSparse" : false, 
                "isPartial" : false, 
                "indexVersion" : 2.0, 
                "direction" : "forward", 
                "indexBounds" : {
                    "item.name" : [
                        "[\"cd\", \"cd\"]"
                    ]
                }
            }
        }, 
        "rejectedPlans" : [

        ]
    }, 
    "executionStats" : {
        "executionSuccess" : true, 
        "nReturned" : 1.0, 
        "executionTimeMillis" : 0.0, 
        "totalKeysExamined" : 1.0, 
        "totalDocsExamined" : 1.0, 
        "executionStages" : {
            "stage" : "FETCH", 
            "nReturned" : 1.0, // 返回的1条记录
            "executionTimeMillisEstimate" : 0.0,  // 执行了时间 
            "works" : 2.0, 
            "advanced" : 1.0, 
            "needTime" : 0.0, 
            "needYield" : 0.0, 
            "saveState" : 0.0, 
            "restoreState" : 0.0, 
            "isEOF" : 1.0, 
            "docsExamined" : 1.0, // 查询了一条document文档
            "alreadyHasObj" : 0.0, 
            "inputStage" : {
                "stage" : "IXSCAN", 
                "nReturned" : 1.0, 
                "executionTimeMillisEstimate" : 0.0, 
                "works" : 2.0, 
                "advanced" : 1.0, 
                "needTime" : 0.0, 
                "needYield" : 0.0, 
                "saveState" : 0.0, 
                "restoreState" : 0.0, 
                "isEOF" : 1.0, 
                "keyPattern" : {
                    "item.name" : 1.0
                }, 
                "indexName" : "item.name_1", 
                "isMultiKey" : false, 
                "multiKeyPaths" : {
                    "item.name" : [

                    ]
                }, 
                "isUnique" : false, 
                "isSparse" : false, 
                "isPartial" : false, 
                "indexVersion" : 2.0, 
                "direction" : "forward", 
                "indexBounds" : {
                    "item.name" : [
                        "[\"cd\", \"cd\"]"
                    ]
                }, 
                "keysExamined" : 1.0, 
                "seeks" : 1.0, 
                "dupsTested" : 0.0, 
                "dupsDropped" : 0.0
            }
        }
    }, 
    "serverInfo" : {
        "host" : "cdcbbd0f125a", 
        "port" : 27017.0, 
        "version" : "4.2.0", 
        "gitVersion" : "a4b751dcf51dd249c5865812b390cfd1c0129c30"
    }, 
    "ok" : 1.0
}
```

## 和mysql的索引优化是一样的
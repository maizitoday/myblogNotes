---
title:       "mongodb-集群和分片"
subtitle:    ""
description: ""
date:        2019-09-16
author:      "麦子"
image:       "https://c.pxhere.com/images/a1/23/147c2090573fcd3fbc38e7b7b5a7-1593571.jpg!d"
tags:        ["mongodb","集群","分片","分片策略"]
categories:  ["Tech" ]
---

[TOC]

**说明：本文来自于<https://www.bilibili.com/video/av36394924/?p=2>视频讲解**

**官方文档：<https://docs.mongodb.com/manual/core/wiredtiger/>  4.2版本**

**转载地址：https://cloud.tencent.com/developer/article/1013044**

# 配置文件参数

官方地址：https://docs.mongodb.com/v2.4/reference/configuration-options/ 

```properties
port=27002
bind_ip=127.0.0.1   
replSet=maizi  // 集群的名称
```

# 集群

## docker-compose.yml

```yml
version: '3'
services:
  rs1:
    image: mongo4.2.0
    container_name: "rs1"
    ports:
      - "27000:27017"
    volumes:
      - $PWD/mongodb/rs1:/data/db
    command: mongod --dbpath /data/db --replSet mongoreplset
  rs2:
    image: mongo4.2.0
    container_name: "rs2"
    ports:
      - "27001:27017"
    volumes:
      - $PWD/mongodb/rs2:/data/db
    command: mongod --dbpath /data/db --replSet mongoreplset
  rs3:
    image: mongo4.2.0
    container_name: "rs3"
    ports:
      - "27002:27017"
    volumes:
      - $PWD/mongodb/rs3:/data/db
    command: mongod --dbpath /data/db --replSet mongoreplset
```

通过执行 docker-compose up -d 启动，

## 查看容器IP

```dockerfile
docker exec m1 cat /etc/hosts


172.18.0.4   r3
172.18.0.3   r2
172.18.0.2   r1
```

![Xnip2019-09-17_02-34-19](/img/Xnip2019-09-17_02-34-19.png)



```js
// 初始化集群
rs.initiate()
// 查看集群配置
rs.conf()
// 向集群中添加成员
rs.add("172.18.0.3:27017")
rs.add("172.18.0.4:27017")

// 查看状态
rs.status()


{ 
    "set" : "mongoreplset", 
    "date" : ISODate("2019-09-16T18:52:03.222+0000"), 
    "myState" : 1.0, 
    "term" : NumberLong(2), 
    "syncingTo" : "", 
    "syncSourceHost" : "", 
    "syncSourceId" : -1.0, 
    "heartbeatIntervalMillis" : NumberLong(2000), 
    "optimes" : {
        "lastCommittedOpTime" : {
            "ts" : Timestamp(1568659918, 1), 
            "t" : NumberLong(2)
        }, 
        "lastCommittedWallTime" : ISODate("2019-09-16T18:51:58.380+0000"), 
        "readConcernMajorityOpTime" : {
            "ts" : Timestamp(1568659918, 1), 
            "t" : NumberLong(2)
        }, 
        "readConcernMajorityWallTime" : ISODate("2019-09-16T18:51:58.380+0000"), 
        "appliedOpTime" : {
            "ts" : Timestamp(1568659918, 1), 
            "t" : NumberLong(2)
        }, 
        "durableOpTime" : {
            "ts" : Timestamp(1568659918, 1), 
            "t" : NumberLong(2)
        }, 
        "lastAppliedWallTime" : ISODate("2019-09-16T18:51:58.380+0000"), 
        "lastDurableWallTime" : ISODate("2019-09-16T18:51:58.380+0000")
    }, 
    "lastStableRecoveryTimestamp" : Timestamp(1568659868, 1), 
    "lastStableCheckpointTimestamp" : Timestamp(1568659868, 1), 
    "members" : [
        {
            "_id" : 0.0, 
            "name" : "e2f54aa4b983:27017", 
            "ip" : "172.18.0.2", 
            "health" : 1.0, 
            "state" : 1.0, 
            "stateStr" : "PRIMARY", 
            "uptime" : 2042.0, 
            "optime" : {
                "ts" : Timestamp(1568659918, 1), 
                "t" : NumberLong(2)
            }, 
            "optimeDate" : ISODate("2019-09-16T18:51:58.000+0000"), 
            "syncingTo" : "", 
            "syncSourceHost" : "", 
            "syncSourceId" : -1.0, 
            "infoMessage" : "", 
            "electionTime" : Timestamp(1568657897, 1), 
            "electionDate" : ISODate("2019-09-16T18:18:17.000+0000"), 
            "configVersion" : 3.0, 
            "self" : true, 
            "lastHeartbeatMessage" : ""
        }, 
        {
            "_id" : 1.0, 
            "name" : "172.18.0.3:27017", 
            "ip" : "172.18.0.3", 
            "health" : 1.0, 
            "state" : 2.0, 
            "stateStr" : "SECONDARY", 
            "uptime" : 1297.0, 
            "optime" : {
                "ts" : Timestamp(1568659918, 1), 
                "t" : NumberLong(2)
            }, 
            "optimeDurable" : {
                "ts" : Timestamp(1568659918, 1), 
                "t" : NumberLong(2)
            }, 
            "optimeDate" : ISODate("2019-09-16T18:51:58.000+0000"), 
            "optimeDurableDate" : ISODate("2019-09-16T18:51:58.000+0000"), 
            "lastHeartbeat" : ISODate("2019-09-16T18:52:02.822+0000"), 
            "lastHeartbeatRecv" : ISODate("2019-09-16T18:52:02.091+0000"), 
            "pingMs" : NumberLong(0), 
            "lastHeartbeatMessage" : "", 
            "syncingTo" : "e2f54aa4b983:27017", 
            "syncSourceHost" : "e2f54aa4b983:27017", 
            "syncSourceId" : 0.0, 
            "infoMessage" : "", 
            "configVersion" : 3.0
        }, 
        {
            "_id" : 2.0, 
            "name" : "172.18.0.4:27017", 
            "ip" : "172.18.0.4", 
            "health" : 1.0, 
            "state" : 2.0, 
            "stateStr" : "SECONDARY", 
            "uptime" : 1293.0, 
            "optime" : {
                "ts" : Timestamp(1568659918, 1), 
                "t" : NumberLong(2)
            }, 
            "optimeDurable" : {
                "ts" : Timestamp(1568659918, 1), 
                "t" : NumberLong(2)
            }, 
            "optimeDate" : ISODate("2019-09-16T18:51:58.000+0000"), 
            "optimeDurableDate" : ISODate("2019-09-16T18:51:58.000+0000"), 
            "lastHeartbeat" : ISODate("2019-09-16T18:52:02.822+0000"), 
            "lastHeartbeatRecv" : ISODate("2019-09-16T18:52:01.643+0000"), 
            "pingMs" : NumberLong(0), 
            "lastHeartbeatMessage" : "", 
            "syncingTo" : "e2f54aa4b983:27017", 
            "syncSourceHost" : "e2f54aa4b983:27017", 
            "syncSourceId" : 0.0, 
            "infoMessage" : "", 
            "configVersion" : 3.0
        }
    ], 
    "ok" : 1.0, 
    "$clusterTime" : {
        "clusterTime" : Timestamp(1568659918, 1), 
        "signature" : {
            "hash" : BinData(0, "AAAAAAAAAAAAAAAAAAAAAAAAAAA="), 
            "keyId" : NumberLong(0)
        }
    }, 
    "operationTime" : Timestamp(1568659918, 1)
}

```

| 容器名 | ip         | 备注       |
| :----- | :--------- | :--------- |
| m0     | 172.18.0.2 | Primary    |
| m1     | 172.18.0.3 | Secondary1 |
| m2     | 172.18.0.4 | Secondary2 |



## 验证同步

![Xnip2019-09-17_03-02-13](/img/Xnip2019-09-17_03-02-13.png)

mongodb默认读写都是在Primary上进行的，副本节点不允许读写，可以使用如下命令来允许副本读：

```js
db.getMongo().setSlaveOk()
```



## 故障转移

副本集模式下，如果Primary不可用，整个集群将会选举出新的Primary来继续对外提供读写服务

## 问题

对于调用mongodb服务的应用来说，应该怎么连接mongodb呢？一开始直连m0，等到出了问题再手工切换到m2么？

## 注意

https://cloud.tencent.com/developer/article/1013044  这篇文章写的很详细，需要的时候直接看此文章。 



# 分片

集群数据冗余性和高可用性， 分片是将大量的操作均摊到各自的服务器上面， 负载均衡，提高系统的吞吐量。 

![Xnip2019-09-17_14-06-31](/img/Xnip2019-09-17_14-06-31.png)

原来我们用replica set 他是使用一台机器承载所有的流量，   现在用分片是我们用三台去承载。 分流了。 



# 分片集群

单一的分片在实际项目中， 无法保证高可用。 所以如果要分片分流的话， 直接使用分片集群处理。 

MongoDB分片[群集](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)包含以下组件：

- [分片](https://docs.mongodb.com/manual/core/sharded-cluster-shards/)：每个分片包含分片数据的子集。每个分片都可以部署为[副本集](https://docs.mongodb.com/manual/reference/glossary/#term-replica-set)。
- [mongos](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/)：`mongos`充当查询路由器，提供客户端应用程序和分片集群之间的接口。
- [config servers](https://docs.mongodb.com/manual/core/sharded-cluster-config-servers/)：配置服务器存储群集的元数据和配置设置。从MongoDB 3.4开始，必须将配置服务器部署为副本集

![Xnip2019-09-17_14-40-19](/img/Xnip2019-09-17_14-40-19.png)

## 1. 配置服务器

config servers：配置服务器存储分片[集群](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)的元数据。元数据反映了分片集群中所有数据和组件的状态和组织。元数据包括每个分片上的块列表以及定义块的范围。

该[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)实例缓存这些数据，并用它来路由读写操作，以正确的碎片。[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 在群集有元数据更改时更新缓存，例如[Chunk Splits](https://docs.mongodb.com/manual/core/sharding-data-partitioning/#sharding-chunk-splits)或[添加分片](https://docs.mongodb.com/manual/tutorial/add-shards-to-shard-cluster/)。Shards还从配置服务器读取块元数据。

配置服务器还存储[身份验证](https://docs.mongodb.com/manual/core/authentication/)配置信息，例如[基于角色的访问控制](https://docs.mongodb.com/manual/core/authorization/)或群集的[内部身份验证](https://docs.mongodb.com/manual/core/security-internal-authentication/)设置。

MongoDB还使用配置服务器来管理分布式锁。

每个分片群集必须具有自己的配置服务器。不要将相同的配置服务器用于不同的分片群集。

生产上建议config server的rs至少要有3个副本集成员。

**MongoDB是在collection级别实现的水平分片。**



## 2. mongos

MongoDB [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)实例将查询和写入操作路由到分[片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)集群中的分片。[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)从应用程序的角度提供分片集群的唯一接口。**应用程序永远不会与分片直接连接或通信**。

一个Sharding集群，可以有一个mongos，也可以如上图所示为每个App Server配置一个mongos以减轻路由压力。

注意这里的mongos并不要配置为rs，因为只是个路由，并不存储数据，配置多个mongos的意思是配置多个单独的mongos实例。



## 3. 分片

从MongoDB 3.6开始，必须将分片部署为[副本集](https://docs.mongodb.com/manual/reference/glossary/#term-replica-set)以提供冗余和高可用性。

分片群集中的每个数据库都有一个[主分片](https://docs.mongodb.com/manual/reference/glossary/#term-primary-shard)，其中包含该数据库的所有非分片集合。每个数据库都有自己的主分片。[主](https://docs.mongodb.com/manual/reference/glossary/#term-primary)分片与副本集中的[主分片](https://docs.mongodb.com/manual/reference/glossary/#term-primary)无关。

用来保存数据，保证数据的高可用性和一致性。可以是一个单独的mongod实例，也可以是一个副本集。在生产环境下Shard一般是一个Replica Set，以防止该数据片的单点故障。可以将所有shard的副本集放在一个服务器多个mongodb实例中。

sharding的每个node的database中的集合可以是分片也可以不分片，每个db都有一个primary shard，未分片的集合就是存在其各自的primary shard中的。



# 分片策略选择

**转载地址： https://www.cnblogs.com/leohahah/p/8652572.html**

## hash sharding

https://docs.mongodb.com/v3.2/core/hashed-sharding/

当shard key总是单调递增时hash sharding并不是一个很好的选择，其查询分发基本和broadcast operation一样了，因为hash会把数据比较均匀的分布在各个shard上，但此时选择ranged sharding也有缺点，因为数据过度集中会导致数据集中于某个shard。

## ranged sharding

https://docs.mongodb.com/v3.2/core/ranged-sharding/

在shard key选取不正确的情况下，范围分片会导致数据分布不均匀，也可能遭遇性能瓶颈，因此需要合理的选择ranged shard key。

## Tag aware sharding

https://docs.mongodb.com/v3.2/core/tag-aware-sharding/

原理如下：

- sh.addShardTag() 给shard设置标签A
- sh.addTagRange() 给集合的某个chunk范围设置标签A，最终MongoDB会保证设置标签 A 的chunk范围（或该范围的超集）分布设置了标签 A 的 shard 上。

Tag aware sharding可应用在如下场景：

将部署在不同机房的shard设置机房标签，将不同chunk范围的数据分布到指定的机房

将服务能力不通的shard设置服务等级标签，将更多的chunk分散到服务能力更强的shard上去。

使用 Tag aware sharding 需要注意是,chunk分配到对应标签的shard上不是立即完成，而是在不断insert、update后触发split、moveChunk后逐步完成的，并且需要保证balancer是开启的。所以你可能会观察到，在设置了tag range后一段时间后，写入仍然没有分布到tag相同的shard上去。
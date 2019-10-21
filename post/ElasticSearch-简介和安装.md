---
title:       "ElasticSearch-入门"
subtitle:    ""
description: ""
date:        2019-09-21
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["elasticSearch","安装","kibana安装"]
categories:  ["Tech" ]
---

[TOC]

**说明：本文来自于https://www.bilibili.com/video/av64033816/?p=1>视频讲解**

**官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/6.0/getting-started.html 版本6.8.3**

# 简介

ElasticSearch是一个基于Apache Luncene的开源的搜索引擎。无论在开源还是专有领域，Lucene可以被认为是迄今为止最先进，性能最好的，功能最齐全的搜索引擎库。

Elasticsearch是一个分布式的文档(document)**存储引擎**。它可以实时存储并检索复杂数据结构——序列化的JSON文档。换言说，一旦文档被存储在Elasticsearch中，它就可以在集群的任一节点上被检索。它的**搜索存储功**能主要是 Lucene 提供的，Lucene 相当于其存储引擎，它在之上封装了索引，查询，以及分布式相关的接口。



## 使用场景

Elasticsearch是当今最流行的分布式搜索引擎，首先介绍下使用场景，比如：

### 全文检索

比如找到与搜索词项(term)最相关的维基百科文章。

### 聚合

比如在广告网络中，可视化的搜索词项的竞价直方图。它允许你在数据上生成复杂的分析统计。它很像SQL中的 GROUP BY 但是功能更强大。

### 地理空间API

比如在顺风车平台，匹配最近的司机和乘客。



## 特点

1. 分布式的实时文件存储，**每个字段都被索引并可被搜索**。 
2. 分布式实时分析搜索引擎，做不规则查询。 
3. 可以扩展到上百台服务器，处理PB级结构化或非结构化数据。
4. 通过简单的RESTFul API来隐藏Lucene的复杂性，从而让全文搜索变的简单。 

他相当于一个单独的服务器， 

##  

## ES能做什么

全文检索(全部字段), 模糊查询(搜索),数据分析(提供分析语法，列如聚合)



## ES 的基本组件

### 索引（index）

文档容器可理解为索引是具有类似属性的文档集合，类似表且索引名须为小写字母；**相当于mysql中的数据库**

### 类型（type）

类型是索引内部的逻辑分区，其意义完全取决于用户需求，一个索引内部可定义一个或多个类型，类型就是其拥有相同的域的文档的预定义,建议一个索引只存一类数据；**相当于mysql中的Tables**

### 文档（document） 

文档是lucene索引的搜索原子单位，它包含了一个或多个域。是域的容器；
每个域的组成部分：一个名字，一个或多个值。拥有多个值的域，通常称为‘多值域’；**相当于mysql中的Rows**

### 映射（mapping）

原始内容存储为文档需要事先分析（如何切词，哪些可以过滤等）分析完后要定义这个分析，定义这个分析后让它怎么去根据这个定义去搜索实现，这个过程就叫映射；
例如：切词、过滤掉某些词等。除此之外ES还为映射提供了诸如将域中的内容排序等功能；

**这个mapping有点类似我们定义MySQL的数据库表结构的时候，需要指定每个字段的名字，其数据类型一样。**

**mapping的目的就是定制化你的字段类型，按照你自己的要求。**

```json
# 设置postdate数据类型
PUT test
{
  "mappings":{
    "my":{
      "properties": {
        "postdate":{
          "type":"date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        }
      }
    }
  }
}

# 添加数据
PUT test/my/1
{
  "postdate":"2018-01-13"
}
PUT test/my/2
{
  "postdate":"2018-01-01 00:01:05"
}
PUT test/my/3
{
  "postdate":"1420077400001"
}
```



# elasticSearch安装成功

[http://localhost:9200/](http://localhost:9200/)

```json
{
    "name": "qLcpGmb",
    "cluster_name": "docker-cluster",
    "cluster_uuid": "4MXOww8BSS-y46oN1cXkdA",
    "version": {
        "number": "6.8.3",
        "build_flavor": "default",
        "build_type": "docker",
        "build_hash": "0c48c0e",
        "build_date": "2019-08-29T19:05:24.312154Z",
        "build_snapshot": false,
        "lucene_version": "7.7.0",
        "minimum_wire_compatibility_version": "5.6.0",
        "minimum_index_compatibility_version": "5.0.0"
    },
    "tagline": "You Know, for Search"
}
```



# kibana-CRUD

```json

#新增
POST my_inde/doc/3
{
  "name": "maizi-today",
  "age": 18
}

#删
DELETE my_inde/doc/3

#查
GET my_inde/_search


#改
PUT my_inde/doc/1
{
  "name": "maizi-today",
  "age": 180
}
```



# 注意

docker安装上面服务，请查看**Dockerfile安装常用环境**文章。 


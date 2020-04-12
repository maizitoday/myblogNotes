---
title:       "ElasticSearch-存储和数据结构"
subtitle:    ""
description: "ElasticSearch存储和数据结构类型介绍"
date:        2019-09-22
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["elasticSearch","存储","数据类型"]
categories:  ["Tech" ]
---

[TOC]

**说明：本文来自于https://www.bilibili.com/video/av64033816/?p=1>视频讲解**

**官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/6.0/getting-started.html 版本6.8.3**

# 存储

Elasticsearch是一个分布式的文档(document)**存储引擎**。它可以实时存储并检索复杂数据结构——序列化的JSON文档。换言说，一旦文档被存储在Elasticsearch中，它就可以在集群的任一节点上被检索。它的**搜索存储功**能主要是 Lucene 提供的，Lucene 相当于其存储引擎，它在之上封装了索引，查询，以及分布式相关的接口。

## 存储方式

### 面向文档

他可以存储整个对象或者文档。然后他不仅仅是存储，还会索引每个文档的内容使之可以被搜索。在es中，你可以对文档进行索引，搜索，排序，过滤。

### 存储大小  

这个没有具体的大小限制的。

# 数据

## 数据结构

使用JSON格式进行存储。 

## 数据类型

原文链接：https://blog.csdn.net/chengyuqiang/article/details/79048800

### 核心数据类型

|      类型      |                             说明                             |
| :------------: | :----------------------------------------------------------: |
|     字符串     | [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/text.html) 和 [`keyword`] |
|  数值数据类型  | long`，`integer`，`short`，`byte`(整数)    `double`，`float`，`half_float`，`scaled_float(浮点) |
|  日期数据类型  |                             date                             |
|  布尔数据类型  |                           boolean                            |
| 二进制数据类型 |                            binary                            |
|  范围数据类型  | integer_range`，`float_range`，`long_range`，`double_range`，`date_range |

#### 字符串

**text**

当一个字段是要被全文搜索的，比如Email内容、产品描述，应该使用text类型。设置text类型以后，字段内容会被分析，在生成倒排索引以前，字符串会被分析器分成一个一个词项。text类型的字段不用于排序，很少用于聚合。

**keyword**

keyword类型适用于索引结构化的字段，比如email地址、主机名、状态码和标签。如果字段需要进行过滤(比如查找已发布博客中status属性为published的文章)、排序、聚合。keyword类型的字段只能通过精确值搜索到。

#### 整数类型

在满足需求的情况下，尽可能选择范围小的数据类型。比如，某个字段的取值最大值不会超过100，那么选择byte类型即可。迄今为止吉尼斯记录的人类的年龄的最大值为134岁，对于年龄字段，short足矣。字段的长度越短，索引和搜索的效率越高。

#### 浮点类型

对于float、half_float和scaled_float,-0.0和+0.0是不同的值，使用term查询查找-0.0不会匹配+0.0，同样range查询中上边界是-0.0不会匹配+0.0，下边界是+0.0不会匹配-0.0。

其中scaled_float，比如价格只需要精确到分，price为57.34的字段缩放因子为100，存起来就是5734
优先考虑使用带缩放因子的scaled_float浮点类型。

#### date类型

日期类型表示格式可以是以下几种：

1. 日期格式的字符串，比如 “2018-01-13” 或 “2018-01-13 12:10:30”

2. long类型的毫秒数( milliseconds-since-the-epoch，epoch就是指UNIX诞生的UTC时间1970年1月1日0时0分0秒)

3. integer的秒数(seconds-since-the-epoch)

ElasticSearch 内部会将日期数据转换为UTC，并存储为milliseconds-since-the-epoch的long型整数。

#### boolean

逻辑类型（布尔类型）可以接受true/false/”true”/”false”值

#### binary类型

二进制字段是指用base64来表示索引中存储的二进制数据，可用来存储二进制形式的数据，例如图像。默认情况下，该类型的字段**只存储不索引**。二进制类型只支持index_name属性。

#### array类型

在ElasticSearch中，没有专门的数组（Array）数据类型，但是，在默认情况下，任意一个字段都可以包含0或多个值，这意味着每个字段默认都是数组类型，**只不过，数组类型的各个元素值的数据类型必须相同**。在ElasticSearch中，数组是开箱即用的（out of box），不需要进行任何配置，就可以直接使用。
在同一个数组中，数组元素的数据类型是相同的，ElasticSearch不支持元素为多个数据类型：[ 10, “some string” ]，常用的数组类型是：

1. 字符数组: [ “one”, “two” ]

2. 整数数组: productid:[ 1, 2 ]

3. 对象（文档）数组: “user”:[ { “name”: “Mary”, “age”: 12 }, { “name”: “John”, “age”: 10 }]，ElasticSearch内部把对象数组展开为 {“user.name”: [“Mary”, “John”], “user.age”: [12,10]}

   

### 复杂数据类型

|     类型     |           说明           |
| :----------: | :----------------------: |
| 对象数据类型 | object` 用于单个JSON对象 |
| 嵌套数据类型 | nested` 用于JSON对象数组 |

#### object类型

JSON天生具有层级关系，文档会包含嵌套的对象。

```json
PUT test/my/1
{
  "employee":{
    "age":30,
    "fullname":{
      "first":"hadron",
      "last":"cheng"
    }
  }
}

```



###  地理数据类型[编辑](https://github.com/elastic/elasticsearch/edit/6.0/docs/reference/mapping/types.asciidoc)

|       类型       |              说明               |
| :--------------: | :-----------------------------: |
| 地理位置数据类型 |    geo_point` 纬度/经度积分     |
| 地理形状数据类型 | geo_shape` 用于多边形等复杂形状 |



### 专业数据类型[编辑](https://github.com/elastic/elasticsearch/edit/6.0/docs/reference/mapping/types.asciidoc)

|       类型       |                      说明                       |
| :--------------: | :---------------------------------------------: |
|    IP数据类型    |            [`ip` 用于IPv4和IPv6地址]            |
|   完成数据类型   |          completion` 提供自动完成建议           |
| 令牌计数数据类型 |       token_count` 计算字符串中令牌的数量       |
|  mapper-murmur3  | murmur3` 在索引时计算值的哈希并将其存储在索引中 |
|    渗滤器类型    |             接受来自query-dsl的查询             |
|  join` 数据类型  |         为同一索引内的文档定义父/子关系         |

####  ip类型

ip类型的字段用于存储IPv4或者IPv6的地址

```json
PUT test
{
  "mappings": {
    "my":{
      "properties": {
        "nodeIP":{
          "type": "ip"
        }
      }
    }
  }
}

GET test/_search
{
  "query": {
    "term": {
      "nodeIP": "192.168.0.0/16"
    }
  }
}
```


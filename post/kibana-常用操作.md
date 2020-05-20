---
title:       "kibana常用操作"
subtitle:    ""
description: "kibana定义，ELK介绍，什么是搜索引擎，PB数据概念，常用操作"
date:        2019-09-25
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["elasticSearch","kibana"]
categories:  ["Tech" ]
---

[TOC]

# 定义

Kibana 是一款开源的前端应用程序，其基础是 Elastic Stack，可以为 Elasticsearch 中索引的数据提供搜索和数据可视化功能。

# ELK一般用来做啥

ELK组件在海量日志系统的运维中，可用于解决：

1. 分布式日志数据集中式查询和管理
2. 系统监控，包含系统硬件和应用各个组件的监控
3. 故障排查
4. 安全信息和事件管理
5. 报表功能

ELK组件在大数据运维系统中，主要可解决的问题如下：

1. 日志查询，问题排查，上线检查
2. 服务器监控，应用监控，错误报警，Bug管理
3. 性能分析，用户行为分析，安全漏洞分析，时间管理

# 什么是搜索引擎

所谓搜索引擎，就是根据用户需求与一定算法，运用特定策略从互联网检索出制定信息反馈给用户的一门检索技术。

# PB级别数据

PB是数据存储容量的单位，它等于2的50次方个字节，或者在数值上大约等于1000个TB

```java
1TB=1024GB
1GB=1024MB
1MB=1024KB
1KB=1024BYTE
1BYTE=8BIT(二进制位）
```

# kibana基本操作

https://www.jianshu.com/p/f22d869c1fb4
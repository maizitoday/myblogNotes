---
title:       "mongodb-存储引擎"
subtitle:    ""
description: ""
date:        2019-09-16
author:      "麦子"
image:       "https://c.pxhere.com/images/a1/23/147c2090573fcd3fbc38e7b7b5a7-1593571.jpg!d"
tags:        ["mongodb","WiredTiger"]
categories:  ["Tech" ]
---

[TOC]



**说明：本文来自于<https://www.bilibili.com/video/av36394924/?p=2>视频讲解**

**官方文档：<https://docs.mongodb.com/manual/core/wiredtiger/>  4.2版本**

# WiredTiger

从MongoDB 3.2开始，WiredTiger存储引擎是默认的存储引擎。 



## 文档级并发

WiredTiger使用*文档级*并发控制进行写操作。因此，多个客户端可以同时修改集合的不同文档。对于大多数读写操作，WiredTiger使用**乐观并发控制**。



## 快照和检查点

在操作开始时，WiredTiger为操作提供数据的时间点快照。**快照**显示内存中数据的一致视图。写入磁盘时，WiredTiger会以一致的方式将快照中的所有数据写入磁盘，并覆盖所有数据文件。现在[持久的](https://docs.mongodb.com/manual/reference/glossary/#term-durable) 数据充当数据文件中的*检查点*。该*检查点*保证了数据文件直到并包括最后一个检查点相一致; 即检查点可以作为恢复点。

从版本3.6开始，MongoDB配置WiredTiger以60秒的间隔创建检查点（即将快照数据写入磁盘）。



## 日志

WiredTiger将预写日志（即日志）与[检查点](https://docs.mongodb.com/manual/core/wiredtiger/#storage-wiredtiger-checkpoints)结合使用 以确保数据持久性。

WiredTiger日志会在检查点之间保留所有数据修改。如果MongoDB在检查点之间退出，它将使用日志重播自上一个检查点以来修改的所有数据。



## 压缩

使用WiredTiger，MongoDB支持对所有集合和索引进行压缩。压缩可以以额外的CPU为代价最大限度地减少存储使用。



## 内存使用

使用WiredTiger，MongoDB同时使用WiredTiger内部缓存和文件系统缓存。

从MongoDB 3.4开始，默认的WiredTiger内部缓存大小是以下两者中的较大者：

- 50％（RAM - 1 GB），或
- 256 MB。

例如，在总共4GB RAM的系统上，WiredTiger缓存将使用1.5GB的RAM（）。相反，具有总共1.25 GB RAM的系统将为WiredTiger缓存分配256 MB，因为这超过总RAM的一半减去1千兆字节（）。`0.5 * (4 GB - 1GB) = 1.5 GB``0.5 * (1.25 GB - 1 GB) = 128 MB < 256 MB`


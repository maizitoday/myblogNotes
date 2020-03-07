---
title:       "JMS理解"
subtitle:    ""
description: ""
date:        2020-03-02
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java相关概念", "JMS"]
categories:  ["Tech" ]
---

**转载地址：https://www.cnblogs.com/Zender/p/9098410.html**

# 消息服务

消息服务指的是两个应用程序之间进行异步通信的API，它为标准消息协议和消息服务提供了一组通用接口，包括创建、发送、读取消息等，用于支持应用程序开发。在Java中，当两个应用程序使用JMS进行通信时，它们之间并不是直接相连的，而是通过一个共同的消息收发服务连接起来，可以达到解耦的效果。

# 简介

JMS即Java消息服务（Java Message Service）应用程序接口，是一个Java平台中关于面向消息中间件（MOM-分布式系统的集成）的API，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。

JMS是一种与厂商无关的 API，用来访问消息收发系统消息，它类似于JDBC(Java Database Connectivity)。

# 体系架构

JMS由以下元素组成：

| JMS提供者 | 连接面向消息中间件的，JMS接口的一个实现。提供者可以是Java平台的JMS实现，也可以是非Java平台的面向消息中间件的适配器。 |
| :-------: | :----------------------------------------------------------- |
|  JMS客户  | 生产或消费基于消息的Java的应用程序或对象。                   |
| JMS生产者 | 创建并发送消息的JMS客户。                                    |
| JMS消费者 | 接收消息的JMS客户。                                          |
|  JMS消息  | 包括可以在JMS客户之间传递的数据的对象。                      |
|  JMS队列  | 一个容纳那些被发送的等待阅读的消息的区域。与队列名字所暗示的意思不同，消息的接受顺序并不一定要与消息的发送顺序相同。一旦一个消息被阅读，该消息将被从队列中移走。 |
|  JMS主题  | 一种支持发送消息给多个订阅者的机制。                         |

# 通常会说到一些术语

消息中间件（JMS Provider) ： **指提供了对JMS协议的第三方组件，比如RocketMQ就是一个消息中间件，另外比较知名的还有KafKa、 Rabbit MQ、ActiveMQ等**。

消息（Message): 通信内容的载体，其结构主要分为消息头，属性和消息体，并且根据存储结构的不同分为好几种，后面会详细提到。

消息模式：分为点对点（Point to Point，即P2P）和发布/订阅（Pub/Sub)，对应的数据结构分别是队列（Queue)和主题（Topic)

消息生产者：产生消息的一方，在P2P模式下，指消息发送者(Sender)，在P/S模式下指消息发布者(Publisher)

消息消费者：接收消息的一方，对应于两种模式分别是消息接收者（Receiver）和消息订阅者(Subscriber)


# 和RocketMQ关系

指提供了对JMS协议的第三方组件，比如RocketMQ就是一个消息中间件，另外比较知名的还有KafKa、 Rabbit MQ、ActiveMQ。
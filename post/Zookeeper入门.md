---
title:       "Zookeeper-入门"
subtitle:    ""
description: ""
date:        2019-04-23
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-4-25-istio-auto-injection-with-webhook/lion.jpg"
tags:        ["服务注册", "Zookeeper"]
categories:  ["Tech" ]
---

[TOC]



**说明： 本学习主要对尚硅谷大数据研发部《zookeeper视频》记录，一下文字来源这个视频PPT**

**注意：** **Zookeeper 版本 3.4**

# Zookeeper入门

zookeeper是一个开源的分布式的，为分布式应用**提供协调服务**的项目。 



## 工作机制

![4-26-1](/img/4-26-1.png)



## 特点

![4-26-2](/img/4-26-2.png)





## 数据结构

![4-26-3](/img/4-26-3.png)



## 应用场景

### 统一命名服务

![4-26-4](/img/4-26-4.png)



### 统一配置管理

![4-26-5](/img/4-26-5.png)



### 统一集群管理

![4-26-6](/img/4-26-6.png)



### 服务器动态上下线

![4-26-7](/img/4-26-7.png)



### 软负载均衡

![4-26-8](/img/4-26-8.png)



# Zookeeper集群安装

## 创建docker-compose.yml

```yaml
version: '3.4'

services:
  zoo1:
    image: zookeeper
    restart: always
    hostname: zoo1
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

  zoo2:
    image: zookeeper
    restart: always
    hostname: zoo2
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=0.0.0.0:2888:3888 server.3=zoo3:2888:3888

  zoo3:
    image: zookeeper
    restart: always
    hostname: zoo3
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=0.0.0.0:2888:3888
```



## 运行查看服务

```yaml
#运行
docker-compose up -d

#查看运行服务 
docker-compose ps 

#如下：
➜  zookeeper docker-compose ps
      Name                    Command               State                     Ports
------------------------------------------------------------------------------------------------------
zookeeper_zoo1_1   /docker-entrypoint.sh zkSe ...   Up      0.0.0.0:2181->2181/tcp, 2888/tcp, 3888/tcp
zookeeper_zoo2_1   /docker-entrypoint.sh zkSe ...   Up      0.0.0.0:2182->2181/tcp, 2888/tcp, 3888/tcp
zookeeper_zoo3_1   /docker-entrypoint.sh zkSe ...   Up      0.0.0.0:2183->2181/tcp, 2888/tcp, 3888/tcp

#进入其中一个容器
➜  zookeeper docker-compose exec zoo1 /bin/bash
```



## 集群搭建修改

用docker-compose进行集群搭建可以看出， 主要是下面这个地方的修改，

```yaml
 environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=0.0.0.0:2888:3888 server.3=zoo3:2888:3888
```

进入容器中查看：

```yaml
#myid 位置和值 
bash-4.4# pwd
/data
bash-4.4# ls
myid       version-2
bash-4.4# cat myid
2
bash-4.4#

#zoo.cfg中配置了server服务组
bash-4.4# cat zoo.cfg
clientPort=2181
dataDir=/data
dataLogDir=/datalog
tickTime=2000
initLimit=5
syncLimit=2
autopurge.snapRetainCount=3
autopurge.purgeInterval=0
maxClientCnxns=60
server.1=zoo1:2888:3888
server.2=0.0.0.0:2888:3888
server.3=zoo3:2888:3888
bash-4.4#
```

如果是Linux中配置，也只需要修改这两个地方。 

其中zoo.cfg的修改就在下载的Zookeeper包，解压里面的conf里面的**zoo.cfg**直接修改。**myid**创建在zoo.cfg配置文件中  dataDir=/data ， 这个路径下。



## 查看服务启动状态

```shell
#查看服务状态 
bash-4.4# pwd
/zookeeper-3.4.14/bin
bash-4.4# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Mode: follower  #这里是follower角色

#启动客户端
bash-4.4# ./zkCli.sh
```



## 工具连接Zookeeper

zookeeper-visualizer这个工具很好用，下载地址：https://github.com/xin497668869/zookeeper-visualizer，效果图如下：

![4-26-9](/img/4-26-9.png)



## zoo.cfg配置文件

转载地址：https://blog.csdn.net/lan12334321234/article/details/70049945 

```properties
clientPort=2181
dataDir=/data
dataLogDir=/datalog
tickTime=2000
initLimit=5
syncLimit=2
autopurge.snapRetainCount=3
autopurge.purgeInterval=0
maxClientCnxns=60
server.1=zoo1:2888:3888
server.2=0.0.0.0:2888:3888
server.3=zoo3:2888:3888

 #2181：对cline端提供服务
 #2888：集群内机器通讯使用（Leader监听此端口），副本数据同步端口。
 #3888：选举leader使用
```



### clientPort

客户端连接的接口，客户端连接zookeeper服务器的端口，zookeeper会监听这个端口，接收客户端的请求访问！这个端口默认是2181。



### dataDir

该属性对应的目录是用来存放myid信息跟一些版本，日志，跟服务器唯一的ID信息等。



### dataLogDir

是事务日志目录



### tickTime

客户端与服务器或者服务器与服务器之间维持心跳，也就是每个tickTime时间就会发送一次心跳。通过心跳不仅能够用来监听机器的工作状态，还可以通过心跳来控制Flower跟Leader的通信时间，默认情况下FL的会话时常是心跳间隔的两倍。



### initLimit

集群中的follower服务器(F)与leader服务器(L)之间**初始连接**时能容忍的最多心跳数（tickTime的数量）



### syncLimit

集群中flower服务器（F）跟leader（L）服务器之间的请求和答应最多能容忍的心跳数。 



### autopurge.purgeInterval

在上文中已经提到，3.4.0及之后版本，ZK提供了自动清理事务日志和快照文件的功能，这个参数指定了清理频率，单位是小时，需要配置一个1或更大的整数，默认是0，表示不开启自动清理功能。



### autopurge.snapRetainCount

这个参数和上面的参数搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个。



### maxClientCnxns

单个客户端与单台服务器之间的连接数的限制，是ip级别的，默认是60，如果设置为0，那么表明不作任何限制。请注意这个限制的使用范围，**仅仅是单台客户端机器与单台ZK服务器之间的连接数限制**，不是针对指定客户端IP，也不是ZK集群的连接数限制，也不是单台ZK对所有客户端的连接数限制。



### server.A=B：C：D：

server.A=B：C：D：其中 A 是一个数字，表示这个是第几号服务器；**B 是这个服务器的 ip 地址**；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。**Zookeeper 启动时会读取dataDir里面的myid文件，拿到里面的数据与 zoo.cfg 里面的配置信息比较从而判断到底是那个 server。**  

 

# Zookeeper内部原理



## 选举机制

![4-26-10](/img/4-26-10.png)



### 全新集群选举

也就是服务依次启动的时候，刚开始启动。

 ![4-26-11](/img/4-26-11.png)

其中需要注意， 当1号机器，收到3号机器发送过来的信息后，他会把投2号的一票，去投3号，2号的也会把自己的一票投3号机器， 相当于一个投票池子里面， 每一台机器只能投一票，当发现比自己myid大的时候，就把投给自己唯一个票投给比他大的myid。 

**注意：在初始启动时候， myid的值越大，机会就大，其他机器认为你比他要牛。**



### 非全新集群选举

![4-26-12](/img/4-26-12.png)



### 脑裂是什么？Zookeeper是如何解决的？

转载地址：https://juejin.im/post/5d36c2f25188257f6a209d37

也就是过半机制的选举Leader。



## 节点类型

![4-26-13](/img/4-26-13.png)



## 监听器的原理

![4-26-16](/img/4-26-16.png)



## 写数据流程

![4-26-17](/img/4-26-17.png)


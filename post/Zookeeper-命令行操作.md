---
title:       "Zookeeper-常用命令"
subtitle:    ""
description: ""
date:        2019-04-26
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-05-23-service_2_service_auth/background.jpg"
tags:        ["nosql", "节点详细说明"]
categories:  ["Tech" ]
---



[TOC]

**说明： 本学习主要对尚硅谷大数据研发部《zookeeper视频》记录，一下文字来源这个视频PPT**

**注意：** **Zookeeper 版本 3.4**

# 命令行操作



## 连接zk集群

```shell
bin/zkCli.sh -server localhost:2181,localhost:2182,localhost:2183
```



## 查看zk集群中谁是leader 和当前zk的状态：

```shell
bin/zkServer.sh status
```



##  ls 

查看当前znode中包含的内容

```shell
[zk: localhost:2181(CONNECTED) 3] ls /
[zookeeper]
```



## ls2

查看当前znode详细数据

```shell
[zk: localhost:2181(CONNECTED) 4] ls2 /
[zookeeper]
cZxid = 0x0
ctime = Thu Jan 01 00:00:00 GMT 1970
mZxid = 0x0
mtime = Thu Jan 01 00:00:00 GMT 1970
pZxid = 0x0
cversion = -1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
```



## create

创建节点， 如果原来没有序号节点，序号从0开始一次递增，如果原节点下已有2个节点，则再排序时从2开始，依次类推

```shell
[zk: localhost:2181(CONNECTED) 6] create /flag success  #注意需要写值，不然失败
Created /flag
[zk: localhost:2181(CONNECTED) 7] ls /
[flag, zookeeper]

选择
-e : 短暂节点
-s:  序列节点
默认是持久节点。  create -e -s  /flag success
```



## get

获取节点值

```shell
[zk: localhost:2181(CONNECTED) 8] get /flag
success
cZxid = 0x20000001e
ctime = Fri Apr 26 14:02:15 GMT 2019
mZxid = 0x20000001e
mtime = Fri Apr 26 14:02:15 GMT 2019
pZxid = 0x20000001e
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 0
```



## set

修改数据节点

```shell
[zk: localhost:2181(CONNECTED) 4] set /flag  maizi
cZxid = 0x20000001e
ctime = Fri Apr 26 14:02:15 GMT 2019
mZxid = 0x20000002a
mtime = Fri Apr 26 14:16:14 GMT 2019
pZxid = 0x20000001e
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
```



## get path [watch]

监听值的变化

2181窗口

```shell
[zk: localhost:2181(CONNECTED) 10] get /flag watch
maizi
cZxid = 0x20000001e
ctime = Fri Apr 26 14:02:15 GMT 2019
mZxid = 0x20000002f
mtime = Fri Apr 26 14:23:43 GMT 2019
pZxid = 0x20000001e
cversion = 0
dataVersion = 5
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
[zk: localhost:2181(CONNECTED) 11]
WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/flag #这里就监控到了
```

```shell
[zk: localhost:2181(CONNECTED) 7] set /flag xiaoqiang
cZxid = 0x20000001e
ctime = Fri Apr 26 14:02:15 GMT 2019
mZxid = 0x200000030
mtime = Fri Apr 26 14:24:06 GMT 2019
pZxid = 0x20000001e
cversion = 0
dataVersion = 6
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 9
numChildren = 0
```

**注意：每一次watch，只能监控一次， 下次在set操作，不会在进行监控。**



## ls path watch 

监听路径变化，和上面的一样，只监控一次。 

```shell
WATCHER::

WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/flag
```



## delete path

删除节点

```shell
[zk: localhost:2181(CONNECTED) 8] delete /ok0000000003
[zk: localhost:2181(CONNECTED) 9] ls /
[flag, zookeeper]
```



## rmr  path 

递归删除节点

```shell
[zk: localhost:2181(CONNECTED) 12] ls /
[flag, zookeeper]
[zk: localhost:2181(CONNECTED) 13] delete /flag
Node not empty: /flag
[zk: localhost:2181(CONNECTED) 14] rmr /flag
[zk: localhost:2181(CONNECTED) 15] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 16]
```



## stat path 

查看节点状态 

```shell
[zk: localhost:2181(CONNECTED) 16] stat /zookeeper
cZxid = 0x0
ctime = Thu Jan 01 00:00:00 GMT 1970
mZxid = 0x0
mtime = Thu Jan 01 00:00:00 GMT 1970
pZxid = 0x200000019
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 3
```



# stat结构体

![4-26-14](/img/4-26-14.png)

![4-26-15](/img/4-26-15.png)
---
title:       "面试题系列-Redis"
subtitle:    ""
description: "缓存穿透，缓存雪崩, LRU算法"
date:        2020-06-03
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["面试题系列","redis"]
categories:  ["Tech" ]
---

[TOC]

**说明：以下文字来源网络各相关面试题目收集**

# 什么是 Redis？简述它的优缺点？

Redis 的全称是：Remote Dictionary.Server，本质上是一个 Key-Value 类型的内存数据库，很像memcached，整个数据库统统加载在内存当中进行操作，定期通过异步操作把数据库数据 flush 到硬盘上进行保存。因为是纯内存操作，Redis 的性能非常出色，每秒可以处理超过 10 万次读写操作，是已知性能最快的Key-Value DB。

Redis 的出色之处不仅仅是性能，Redis 最大的魅力是支持保存多种数据结构，此外单个 value 的最大限制是 1GB，不像 memcached 只能保存 1MB 的数据，因此 Redis 可以用来实现很多有用的功能。比方说用他的 List 来做 FIFO 双向链表，实现一个轻量级的高性 能消息队列服务，用他的 Set 可以做高性能的 tag 系统等等。

另外 Redis 也可以对存入的 Key-Value 设置 expire 时间，因此也可以被当作一 个功能加强版的memcached 来用。 

Redis 的主要缺点是数据库容量受到物理内存的限制，不能用作海量数据的高性能读写，因此 Redis 适合的场景主要局限在较小数据量的高性能操作和运算上。



# 一个字符串类型的值能存储最大容量是多少？

512M



# 为什么 Redis 需要把所有数据放到内存中？

Redis 为了达到最快的读写速度将数据都读到内存中，并通过异步的方式将数据写入磁盘。所以 redis 具有快速和数据持久化的特征，如果不将数据放在内存中，磁盘 I/O 速度为严重影响 redis 的性能。

在内存越来越便宜的今天，redis 将会越来越受欢迎， 如果设置了最大使用的内存，则数据已有记录数达到内存限值后不能继续插入新值。



# 什么是缓存穿透？如何避免？什么是缓存雪崩？何如避免？



## 缓存穿透

一般的缓存系统，都是按照 key 去缓存查询，如果不存在对应的 value，就应该去后端系统查找（比如DB）。一些恶意的请求会故意查询不存在的 key,请求量很大，就会对后端系统造成很大的压力。这就叫做缓存穿透。

### 如何避免？

1：对查询结果为空的情况也进行缓存，缓存时间设置短一点，或者该 key 对应的数据 insert 了之后清理缓存。

2：对一定不存在的 key 进行过滤。可以把所有的可能存在的 key 放到一个大的 Bitmap 中，查询时通过该 bitmap 过滤。



## 缓存雪崩

当缓存服务器重启或者大量缓存集中在某一个时间段失效，这样在失效的时候，会给后端系统带来很大压力。导致系统崩溃。

### 如何避免？

1：在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个 key 只允许一个线程查询数据和写缓存，其他线程等待。

2：做二级缓存，A1 为原始缓存，A2 为拷贝缓存，A1 失效时，可以访问 A2，A1 缓存失效时间设置为短期，A2 设置为长期

3：不同的 key，设置不同的过期时间，让缓存失效的时间点尽量均匀



# 为什么高并发下有时单线程的 redis 比多线程的 memcached 效率要高？

区别：

1. mc 可缓存图片和视频。rd 支持除 k/v 更多的数据结构;
2. rd 可以使用虚拟内存，rd 可持久化和 aof 灾难恢复，rd 通过主从支持数据备份;
3. rd 可以做消息队列。
4. 存储的内容比较大，memcache 存储的 value 最大为 1M。

**原因：mc 多线程模型引入了缓存一致性和锁，加锁带来了性能损耗。**



# 知道 redis 的持久化吗？底层如何实现的？有什么优点缺点？

RDB(Redis DataBase:在不同的时间点将 redis 的数据生成的快照同步到磁盘等介质上):内存 到硬盘的快照，定期更新。缺点：耗时，耗性能(fork+io 操作)，易丢失数据。

AOF(Append Only File：将 redis 所执行过的所有指令都记录下来，在下次 redis 重启时，只 需要执行指令就可以了):写日志。缺点：体积大，恢复速度慢。

bgsave 做镜像全量持久化，aof 做增量持久化。因为 bgsave 会消耗比较长的时间，不够实 时，在停机的时候会导致大量的数据丢失，需要 aof 来配合，在 redis 实例重启时，优先使 用 aof 来恢复内存的状态，如果没有 aof 日志，就会使用 rdb 文件来恢复。Redis 会定期做 aof 重写，压缩 aof 文件日志大小。Redis4.0 之后有了混合持久化的功能，将 bgsave 的全量 和 aof 的增量做了融合处理，这样既保证了恢复的效率又兼顾了数据的安全性。bgsave 的 原理，fork 和 cow, fork 是指 redis 通过创建子进程来进行 bgsave 操作，cow 指的是 copy on write，子进程创建后，父子进程共享数据段，父进程继续提供读写服务，写脏的页面数据 会逐渐和子进程分离开来。



# redis 过期策略都有哪些？LRU 算法知道吗？

过期策略:

定时过期(一 key 一定时器)，惰性过期：只有使用 key 时才判断 key 是否已过期，过期则清 除。定期过期：前两者折中。

LRU:

```java
new LinkedHashMap<K, V>(capacity, DEFAULT_LOAD_FACTORY, true);//第三个参数置为 true，
```

代表 linkedlist 按访问顺序排序，可作为 LRU 缓存；设为 false 代表 按插入顺序排序，可作为 FIFO 缓存

LRU 算法实现：

1. 通过双向链表来实现，新数据插入到链表头部；
2. 每当缓存命中（即缓存 数据被访问），则将数据移到链表头部；
3. 当链表满的时候，将链表尾部的数据丢弃。

LinkedHashMap：HashMap 和双向链表合二为一即是 LinkedHashMap。HashMap 是无序 的，LinkedHashMap 通过维护一个额外的双向链表保证了迭代顺序。该迭代顺序可以是插 入顺序（默认），也可以是访问顺序。



# 缓存与数据库不一致怎么办 

假设采用的主存分离，读写分离的数据库，

如果一个线程 A 先删除缓存数据，然后将数据写入到主库当中，这个时候，主库和从库同 步没有完成，线程 B 从缓存当中读取数据失败，从从库当中读取到旧数据，然后更新至缓 存，这个时候，缓存当中的就是旧的数据。

发生上述不一致的原因在于，主从库数据不一致问题，加入了缓存之后，主从不一致的时 间被拉长了.

处理思路：在从库有数据更新之后，将缓存当中的数据也同时进行更新，即当从库发生了 数据更新之后，向缓存发出删除，淘汰这段时间写入的旧数据。



# Redis 常见的性能问题和解决方案

1. master 最好不要做持久化工作，如 RDB 内存快照和 AOF 日志文件
2. 如果数据比较重要，某个 slave 开启 AOF 备份，策略设置成每秒同步一次
3. 为了主从复制的速度和连接的稳定性，master 和 Slave 最好在一个局域网内
4. 尽量避免在压力大得主库上增加从库
5. 主从复制不要采用网状结构，尽量是线性结构，Master<--Slave1<----Slave2 ....



# Redis 的数据淘汰策略有哪些

1. voltile-lru 从已经设置过期时间的数据集中挑选最近最少使用的数据淘汰
2. voltile-ttl 从已经设置过期时间的数据库集当中挑选将要过期的数据
3. voltile-random 从已经设置过期时间的数据集任意选择淘汰数据
4. allkeys-lru 从数据集中挑选最近最少使用的数据淘汰
5. allkeys-random 从数据集中任意选择淘汰的数据
6. no-eviction 禁止驱逐数据



# 使用 Redis 做过异步队列吗，是如何实现的

使用 list 类型保存数据信息，rpush 生产消息，lpop 消费消息，当 lpop 没有消息时，可 以 sleep 一段时间，然后再检查有没有信息，如果不想 sleep 的话，可以使用 blpop, 在没 有信息的时候，会一直阻塞，直到信息的到来。redis 可以通过 pub/sub 主题订阅模式实现 一个生产者，多个消费者，当然也存在一定的缺点，当消费者下线时，生产的消息会丢 失。



# redis 为什么有16个数据默认库

redis是一个字典结构的存储服务器，一个redis实例提供了多个用来存储数据的字典，客户端可以指定将数据存储在哪个字典中。这与在一个关系数据库实例中可以创建多个数据库类似（如下图所示），所有 可以将其中的每个字典都理解成一个独立的数据库。

redis默认支持16个数据库，可以通过调整redis的配置文件redis/redis.conf中的databases来修改这一个值，设置完毕后重启redis便完成配置。



# 单线程的 redis怎么理解

单线程-多路复用IO模型：处理网络请求和真正的处理都是在同一个也是唯一的一个线程环境中执行的，因此一个慢操作会导致redis的并发量降下来。

具体查看博客[《浅析I/O模型》](https://maizitoday.github.io/post/浅析io模型/#1-阻塞io模型-bio)



# Redis 如何保持和MySQL数据一致

转载地址：https://blog.csdn.net/Thousa_Ho/article/details/78900563?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase 

[canal](https://github.com/alibaba/canal)插件直接配置同步处理。

## 1. MySQL持久化数据,Redis只读数据

redis在启动之后，从数据库加载数据。

读请求：

不要求强一致性的读请求，走redis，要求强一致性的直接从mysql读取

写请求：

数据首先都写到数据库，之后更新redis（先写redis再写mysql，如果写入失败事务回滚会造成redis中存在脏数据）



## 2.MySQL和Redis处理不同的数据类型

MySQL处理实时性数据，例如金融数据、交易数据

Redis处理实时性要求不高的数据，例如网站最热贴排行榜，好友列表等

**在并发不高的情况下**，读操作优先读取redis，不存在的话就去访问MySQL，并把读到的数据写回Redis中；写操作的话，直接写MySQL，成功后再写入Redis(可以在MySQL端定义CRUD触发器，在触发CRUD操作后写数据到Redis，也可以在Redis端解析binlog，再做相应的操作)

**在并发高的情况下**，读操作和上面一样，写操作是异步写，写入Redis后直接返回，然后定期写入MySQL

**说明：上面转载地址有模拟事例说明。**



# mybatis与redis整合

```xml
	<!-- 增加mybatis-redis依赖，这里只有beta版本 -->
	<dependency>
	    <groupId>org.mybatis.caches</groupId>
	    <artifactId>mybatis-redis</artifactId>
	    <version>1.0.0-beta2</version>
	</dependency>
```












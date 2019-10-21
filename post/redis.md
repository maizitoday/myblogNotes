---
title:       "redis系列-入门"
subtitle:    ""
description: ""
date:        2019-04-06
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["redis", "主从复制-redis", "集群-redis", "分布式锁"]
categories:  ["Tech" ]
---

[TOC]

说明： 基于腾讯课堂，https://ke.qq.com/course/389539 这套视频，笔记收集。

### redis基础(version=redis5.0)

####  redis.conf 常用配置

  转载地址： https://www.cnblogs.com/leoxuan/articles/9053514.html

```yaml
#redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程：
daemonize no

#当redis以守护进程方式运行时，redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定：

pidfile /var/run/redis.pid

#指定redis监听端口，默认端口号为6379，作者在自己的一篇博文中解析了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利女歌手Alessia Merz的名字：

port 6379

#设置tcp的backlog，backlog是一个连接队列，backlog队列总和=未完成三次握手队列+已完成三次握手队列。在高并发环境下你需要一个高backlog值来避免慢客户端连接问题。注意Linux内核会将这个值减小到/proc/sys/net/core/somaxconn 的值，所以需要确认增大somaxconn和tcp_max_syn_backlog两个值来达到想要的

效果：

tcp-backlog 511

#绑定的主机地址：

bind 127.0.0.1

#当客户端闲置多长时间后关闭连接，如果指定为0，表示永不关闭：

timeout 300

#设置检测客户端网络中断时间间隔，单位为秒，如果设置为0，则不检测，建议设置为60：

tcp-keepalive 0

#指定日志记录级别，redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose：

loglevel verbose

#日志记录方式，默认为标准输出，如果配置redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null：

logfile stdout

#设置数据库数量，默认值为16，默认当前数据库为0，可以使用select<dbid>命令在连接上指定数据库id：

databases 16

#指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合：

save <seconds><changes>
save 300 10：表示300秒内有10个更改就将数据同步到数据文件

#指定本地数据库文件名，默认值为dump.rdb：

dbfilename dump.rdb

#指定本地数据库存放目录：

dir ./

#设置当本机为slave服务时，设置master服务的IP地址及端口，在redis启动时，它会自动从master进行数据同步：

slaveof <masterip><masterport>

#当master服务设置了密码保护时，slave服务连接master的密码：

masterauth <master-password>

#设置redis连接密码，如果配置了连接密码，客户端在连接redis时需要通过auth <password>命令提供密码，默认关闭：

requirepass foobared

#设置同一时间最大客户端连接数，默认无限制，redis可以同时打开的客户端连接数为redis进程可以打开的最大文件描述符数，如果设置maxclients 0，表示不作限制。当客 户端连接数到达限制时，redis会关闭新的连接并向客户端返回 max number of clients reached错误消息：

maxclients 128

#指定redis最大内存限制，redis在启动时会把数据加载到内存中，达到最大内存后，redis会先尝试清除已到期或即将到期的key，当次方法处理后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制， 会把key存放内存，value会存放在swap区：

maxmemory <bytes>

#设置缓存过期策略，有6种选择：（LRU算法最近最少使用）

volatile-lru：使用LRU算法移除key，只对设置了过期时间的key；

allkeys-lru：使用LRU算法移除key，作用对象所有key；

volatile-random：在过期集合key中随机移除key，只对设置了过期时间的key;

allkeys-random：随机移除key，作用对象为所有key；

volarile-ttl：移除哪些ttl值最小即最近要过期的key；

noeviction：永不过期，针对写操作，会返回错误信息。

maxmemory-policy noeviction

#指定是否在每次更新操作后进行日志记录，redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内数据丢失。因为redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内置存在于内存中。默认为no：

appendonly no

#指定更新日志文件名，默认为appendonly.aof：

appendfilename appendonly.aof

#指定更新日志条件，共有3个可选值：

no：表示等操作系统进行数据缓存同步到磁盘（快）；

always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）；

everysec：表示每秒同步一次（折中，默认值）

appendfsync everysec

#指定是否启用虚拟内存机制，默认值为no，简单介绍一下，VM机制将数据分页存放，由redis将访问量较小的页即冷数据 swap到磁盘上，访问多的页面由磁盘自动换出到内存中：

vm-enabled no

#虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个redis实例共享：

vm-swap-file /tmp/redis.swap

#将所有大于vm-max-memory的数据存入虚拟内存，无论vm-max-memory设置多小，所有索引数据都是内存存储的（redis的索引数据就是keys），也就是说，当vm-max-memory设置为0的时候，其实是所有value都存在于磁盘。默认值为 0：

vm-max-memory 0

#redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是根据存储的数据大小来设定的，作者建议如果储存很多小对象，page大小最好设置为32或者64bytes；如果存储很多大对象，则可以使用更大的page，如果不确定，就使用默认值：

vm-page-size 32

#设置swap文件中page数量，由于页表（一种表示页面空闲或使用的bitmap）是放在内存中的，在磁盘上每8个pages将消耗1byte的内存：

vm-pages 134217728

#设置访问swap文件的线程数，最好不要超过机器的核数，如果设置为0，那么所有对swap文件的操作都是串行的，可能会造成长时间的延迟。默认值为4：

vm-max-threads 4

#设置在客户端应答时，是否把较小的包含并为一个包发送，默认为开启：

glueoutputbuf yes

#指定在超过一定数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法：

hash-max-zipmap-entries 64

hash-max-zipmap-value 512

#指定是否激活重置hash，默认开启：

activerehashing yes

#指定包含其他配置文件，可以在同一主机上多个redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件：

include /path/to/local.conf

```

**注意**：配置redis的时候出现了设置密码而不生效的问题，redis客户端命令行下，执行 config set requirepass 123，然后在重启。(我就是这个问题一直连接不上客户端工具) 



#### docker 安装启动redis

```yaml
docker run 
-p 6379:6379 \
-v /docker_data/redis/6379/data:/data \
-v /docker_data/redis/6379/redis.conf:/etc/redis/redis.conf \
--privileged=true \
--name redis-master \
-d redis-master  redis-server /etc/redis/redis.conf --appendonly yes
```



#### redis可执行文件

```yaml
#cd /usr/local/bin  列出了redis的一些可执行文件
redis-benchmark: 压力测试工具
redis-check-aof: 检查AOF备份文件
redis-check-rdb: 检查RDB备份文件(即RDB)
redis-cli: 客户端工具
redis-sentinel: redis哨兵，通过它实现Redis的HA高可用，用它来监视Master节点
redis-server: redis服务器
```



### redis基本操作

#### String字符串操作类型

```yaml
#set指令
命令： set <key> <value>
说明： 设置key对应的value值，再次执行表示修改。

#get指令
命令： get <key>
说明： 获取key对应的value值

#setnx指令
命令： setnx <key> <value>
说明:  设置key对应的值为string类型的value，如果key存在就返回0，不存在返回1， nx表示 no exists
       如果已经存在了，这个命令是无法改变这个value值的

#incrby指令
命令:  incrby <key>
说明:  对key值做加加操作，并返回值， 如： INCRBY number  2   每次加2

#decrby指令
命令:  decrby <key>
说明:  对key值做减减操作，并返回值，如： DECRBY number  2    每次减2
```



#### Hashes哈希类型操作

```yaml
#hset指令
命令: hset <hash> <filed> <value>
说明: 设置hash中的field对应的value值

#hget指令
命令: hget <hash> <filed>
说明: 获取hash对应的filed中的value值

#相当于集合里面的放入了map
```



####  List列表类型操作

```yaml
#rpush指令
命令: rpush <key> <value>
说明: 在key对应list的尾部添加字符串元素

#lrange指令
命令: lrange <key> <start> <stop>
说明: 根据key查询list中对象，但是需要指定开始行与结束行


```



#### Sets集合类型操作

```yaml
#sadd指令
命令: sadd <key> <value>
说明: 向名为key的set中添加value

#smembers指令
命令: smermbers <keys>
说明: 查看set中的集合
```



### redis常用命令

```yaml
#### kyes指令
命令: kyes *
说明: 返回所有keys对象 

#### exists指令
命令: exists <key>
说明: 确认一个key是否存在 ，如果存在返回1， 不存在返回0 

#### del指令
命令: del <key>
说明: 删除一个key， 如果删除成功返回1， 不存在返回0

#### expire指令
命令: expire <key> <秒>
说明: 设置一个key的过期时间，单位<秒>

#### ttl指令
命令: ttl <key>
说明: 获取key的有效时长，如为-1表示过期 

#### select指令
命令: select <0-15>
说明: 指定数据库，redis默认0-15，共16个数据库 

#### info指令
命令: info
说明: 获取服务器的信息

#### flushdb/flushall指令
命令: flushdb 表示删除当前选择的数据库所有key
命令: flushall 表示删除所有数据库中所有的key
```



### redis事物管理

redis通过mulit，exec，watch等命令来实现事物功能，他将多个命令请求打包，然后一次性按顺序去执行，注意事物执行期间，服务器不会中断事物而去执行其他客户端请求命令，要等事物中的所有命令执行完毕后，才去执行其他客户端的请求命令。

```yaml
#事物开启
#执行  mulit命令
.....................//你操作的命令放入队列
#执行提交事物  exec 

```



#### 锁的原理(watch命令)

![屏幕快照 2019-04-03 下午5.21.26](/img/4-7-1.png)



### redis消息机制

#### publish,subscribe,psubscribe

![屏幕快照 2019-04-03 下午5.53.10](/img/4-7-2.png)



### redis持久化RDB和AOF

RDB 持久化可以在指定的时间间隔内生成数据集的时间点快照，内存。AOF 持久化记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集。Redis 还可以同时使用 AOF 持久化和 RDB 持久化。 在这种情况下， 当 Redis 重启时， 它会优先使用 AOF 文件来还原数据集， 因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整。

#### 优缺点

##### RDB 

持久化能够快速地储存和回复数据， 但是在服务器停机时却会丢失大量数据；

##### AOF

持久化能够有效地提高数据的安全性， 但是在储存和恢复数据方面却要耗费大量的时间。

##### 混合持久化

Redis 还可以同时使用 AOF 持久化和 RDB 持久化。 在这种情况下， 当 Redis 重启时， 它会优先使用 AOF 文件来还原数据集， 因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整。4.0版本的混合持久化默认关闭的，通过aof-use-rdb-preamble配置参数控制，yes则表示开启，no表示禁用，默认是禁用的，可通过config set修改。

优点：a. 混合持久化结合了RDB持久化 和 AOF 持久化的优点, 由于绝大部分都是RDB格式，加载速度快，同时结合AOF，增量的数据以AOF方式保存了，数据更少的丢失。

缺点：b. 兼容性差，一旦开启了混合持久化，在4.0之前版本都不识别该aof文件，同时由于前部分是RDB格式，阅读性较差



### 主从复制

```yaml
docker run \
-p 6379:6379 \
-p 6380:6380 \
-v /docker_data/redis/slave-master-network/data:/data \
-v /docker_data/redis/slave-master-network/redis-6379.conf:/usr/local/bin/redis-6379.conf \
-v /docker_data/redis/slave-master-network/redis-6380.conf:/usr/local/bin/redis-6380.conf \
-v /docker_data/redis/slave-master-network/sentinel.conf:/usr/local/bin/sentinel.conf \
--privileged=true \
--name redis-slave-master \
-d redis-slave-master-by-network  redis-server /usr/local/bin/redis-6380.conf

同时记得把主节点的 rbd和aof关闭， 他不做存储。
```

**注意redis.conf里面的配置（从节点）**
replicaof <masterip> <masterport>：配置主服务的ip和端口。配置之后，就是这台机器的小弟了。主服务也能知道谁是他的小弟。
masterauth <master-password>：如果主服务需要密码认证，这里需要配置从服务连接主服务的密码。
replica-read-only：默认为yes，配置从服务默认为只读模式。

**通过INFO replication命令可以查看这三个服务器的具体信息，如下**

```yaml
# Replication
role:master
connected_slaves:2
slave0:ip=172.17.0.1,port=6381,state=online,offset=3828,lag=1
slave1:ip=172.17.0.1,port=6380,state=online,offset=3828,lag=1
master_replid:0c4bb6fde392b72a61d65cc5d00e895bc77c3f23
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:3828
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:3828
```



### 哨兵机制

为了实现redis故障转移的自动化。自动发现，自动转移。不需要人工参与。当主节点挂掉了， 哨兵去从节点中选择一个从节点来做主节点，当原来的主节点恢复了，他就做从节点了。

sentinel.conf配置(这个服务不要和redis服务部署在一台服务器上，不然redis挂了，他也挂了)

```yaml
#配置一个哨兵 
sentinel monitor mymaster 192.168.70.102 6379 1

#主节点密码
sentinel auth-pass mymaster 123

#默认30秒后，主节点没有心跳， 就认为主机挂掉了。
sentinel down-after-milliseconds mymaster 30000

#2个从节点
sentinel parallel-syncs mymaster 2

#失败切换时候， 允许最大时间
sentinel failover-timeout mymaster 180000

要想让Redis的sentinel（士兵守护）进程在后台自动运行，只要在sentinel配置文件里加上 
*daemonize yes*
```

**无语中** ， docker启动两个redis服务，总是显示从redis服务器ip是 172.17.0.1，醉了，醉了， 没有实现效果。docker也是没有完美的，差评，



### 集群

#### 哨兵模式和集群区别

转载文章(https://blog.csdn.net/drdongshiye/article/details/84204392) 写的很好

##### sentinel模式

sentinel的中文含义是哨兵、守卫。也就是说既然主从模式中，当master节点挂了以后，slave节点不能主动选举一个master节点出来，那么我就安排一个或多个sentinel来做这件事，当sentinel发现master节点挂了以后，sentinel就会从slave中重新选举一个master。sentinel模式基本可以满足一般生产的需求，具备高可用性。但是当数据量过大到一台服务器存放不下的情况时，主从模式或sentinel模式就不能满足需求了，这个时候需要对存储的数据进行分片，将数据存储到多个Redis实例中。

##### cluster模式

cluster的出现是为了解决单机Redis容量有限的问题，将Redis的数据根据一定的规则分配到多台机器。cluster可以说是sentinel和主从模式的结合体，通过cluster可以实现主从和master重选功能，因为Redis的数据是根据一定规则分配到cluster的不同机器的，当数据量过大时，可以新增机器进行扩容。



#### cluster搭建

官方提供至少6个节点

##### redis.conf修改

```yaml
#开启集群模式
cluster-enabled yes
#这个需要和端口对应
cluster-config-file nodes-6380.conf
bind 0.0.0.0
port 6380
#后台运行redis
daemonize no
pidfile /var/run/redis_6380.pid
appendonly yes
appendfilename "appendonly-6380.aof"
dir /usr/local/bin/cluster/6381/  #不同的redis服务数据放入到不停的文件夹


#启动容器， 
docker run \
-p 6380:6380 \
-p 6381:6381 \
-p 6382:6382 \
-p 6383:6383 \
-p 6384:6384 \
-p 6385:6385 \
-p 6386:6386 \
-v /docker_data/redis/redis.conf:/etc/redis/redis.conf \
--privileged=true \
-d redis  redis-server /etc/redis/redis.conf

#另外的几个配置文件我用docker cp 命令放入容器redisconf文件中
```

##### start.sh脚本启动，构建集群

```
#!/bin/sh

cd /usr/local/bin/
redis-server redisconf/redis-6381.conf
redis-server redisconf/redis-6382.conf
redis-server redisconf/redis-6383.conf
redis-server redisconf/redis-6384.conf
redis-server redisconf/redis-6385.conf
redis-server redisconf/redis-6386.conf

redis-cli --cluster create 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385 127.0.0.1:6386 --cluster-replicas 1

#如果需要重新构建集群， 就分别把每个服务器的
 appendonly-6381.aof  dump-6381.rdb  nodes-6381.conf全部删除，然后停止服务，
 然后在调用上面的脚本就好了。
```



#### 查看cluster信息

进入客户端:  redis-cli -c -p 6380

查看节点之间关系命令:   CLUSTER NODES  信息如下，可以看到主从关系：

![屏幕快照 2019-04-06 下午9.41.00](/img/4-7-3.png)

停止 6382主节点， 然后在查看集群信息如下： ![屏幕快照 2019-04-06 下午9.59.50](/img/4-7-4.png)

从这里可以看到原来从节点的6385现在变成了主节点。 现在在重启6382节点，查看

![屏幕快照 2019-04-06 下午10.01.45](/img/4-7-5.png)

6382现在已经变成了slave节点了。

同时cluster并没有实现读写分离的特性，见图， 6385是主节点， 其他节点来获取数据的时候，还是从6385来获取这个数据，并没有从他的从节点来获取。 他是通过槽这个概念来处理数据的。

![屏幕快照 2019-04-06 下午10.07.29](/img/4-7-6.png)

停止6385服务，然后在来获取数据，见图是从6384从服务获取.

![屏幕快照 2019-04-06 下午10.18.53](/img/4-7-7.png)

如果同时删除了一对主从， 集群就就失败， (error) CLUSTERDOWN The cluster is down



#### cluster命令

转载地址(https://www.cnblogs.com/kevingrace/p/7910692.html)

```yaml
#集群
cluster info ：打印集群的信息
cluster nodes ：列出集群当前已知的所有节点（ node），以及这些节点的相关信息。

#节点
cluster meet <ip> <port> ：将 ip 和 port 所指定的节点添加到集群当中，让它成为集群的一份子。
cluster forget <node_id> ：从集群中移除 node_id 指定的节点。
cluster replicate <master_node_id> ：将当前从节点设置为 node_id 指定的master节点的slave节点。只能针#对slave节点操作。
cluster saveconfig ：将节点的配置文件保存到硬盘里面。

#槽(slot)
cluster addslots <slot> [slot ...] ：将一个或多个槽（ slot）指派（ assign）给当前节点。
cluster delslots <slot> [slot ...] ：移除一个或多个槽对当前节点的指派。
cluster flushslots ：移除指派给当前节点的所有槽，让当前节点变成一个没有指派任何槽的节点。
cluster setslot <slot> node <node_id> ：将槽 slot 指派给 node_id 指定的节点，如果槽已经指派给
另一个节点，那么先让另一个节点删除该槽>，然后再进行指派。
cluster setslot <slot> migrating <node_id> ：将本节点的槽 slot 迁移到 node_id 指定的节点中。
cluster setslot <slot> importing <node_id> ：从 node_id 指定的节点中导入槽 slot 到本节点。
cluster setslot <slot> stable ：取消对槽 slot 的导入（ import）或者迁移（ migrate）。

#键
cluster keyslot <key> ：计算键 key 应该被放置在哪个槽上。
cluster countkeysinslot <slot> ：返回槽 slot 目前包含的键值对数量。
cluster getkeysinslot <slot> <count> ：返回 count 个 slot 槽中的键 。
```

同时， redis集群一般都是6个节点起步，也就是说需要3个节点同时挂掉了，才会全部挂掉，这个基本上很难，所以，没有看到集群的重启和停止的命令。  



#### 集群总结

转载地址(<https://juejin.im/entry/596343056fb9a06bc340ac15>)

Redis 集群提供了以下两个好处：

​        将数据自动切分（split）到多个节点的能力。

​         当集群中的一部分节点失效或者无法进行通讯时， 仍然可以继续处理命令请求的能力。

##### 集群下线 

 当主从同时下线，集群就会(error) CLUSTERDOWN The cluster is down

1.  使用`redis-cli --cluster fix`命令来修复群集，以便根据每个节点具有权威性的哈希槽来迁移密钥。

2.  最后使用`redis-cli --cluster check`以确保您的群集正常。

   

##### 数据分片

  集群中的每个节点负责处理一部分哈希槽。 举个例子， 一个集群可以有三个哈希槽， 其中：

- 节点 A 负责处理 0 号至 5500 号哈希槽。
- 节点 B 负责处理 5501 号至 11000 号哈希槽。
- 节点 C 负责处理 11001 号至 16384 号哈希槽。

   这种将哈希槽分布到不同节点的做法使得用户可以很容易地向集群中添加或者删除节点，我们只要保证槽点能够迁移到指定的节点，就可以对现在的节点进行添加和删除了。动态扩充。



##### 数据一致性保证

Redis 并不能保证数据的强一致性. 这意味这在实际中集群在特定的条件下可能会丢失写操作：第一个原因是因为集群是用了异步复制. 写操作过程:

1. 客户端向主节点B写入一条命令.
2. 主节点B向客户端回复命令状态.
3. 主节点将写操作复制给他得从节点 B1, B2 和 B3

a. 数据有可能丢失在回复给客户端后，master断掉了，这个时候还没来的急写入。

b. cluster-node-timeout 10000 ，在重启的一段时间,也是主节点服务宕机 从节点顶替上来需要的时间。一般6秒到10秒。


##### 集群停止的原因

转载地址(https://blog.csdn.net/itcastcn/article/details/78995426)

 a: 如果集群任意master挂掉,且当前master没有slave.集群进入fail状态,也可以理解成集群的slot映射[0-16383]不完成时进入fail状态. ps : redis-3.0.0.rc1加入cluster-require-full-coverage参数,默认关闭,打开集群兼容部分失败.

b:如果集群超过半数以上master挂掉，无论是否有slave集群进入fail状态.

 ps:当集群不可用时,所有对集群的操作做都不可用，收到((error) CLUSTERDOWN The cluster is down)错误

 

### redis实现分布式锁

#### 分布式锁应用场景

转载地址(https://blog.csdn.net/lemon89/article/details/52796775)

某服务提供一组任务，A请求随机从任务组中获取一个任务；B请求随机从任务组中获取一个任务。
在理想的情况下，A从任务组中挑选一个任务，任务组删除该任务，B从剩下的的任务中再挑一个，任务组删除该任务。
同样的，在真实情况下，如果不做任何处理，可能会出现A和B挑中了同一个任务的情况。



#### 分布式锁一般实现三种方式

a.  数据库乐观锁(不建议，性能很差)

b.  基于redis的分布式锁

c.  基于zookeep的分布式锁(一般用这种)

 

#### 分布式锁可用，满足4个条件 



![屏幕快照 2019-04-06 下午11.52.33](/img/4-7-8.png)



#### redis分布式锁原理和步骤

redis的分布式锁实现原理： 控制一个公共资源，也就是redis集群里面的一个key，然后观察这个key的变化，通过setnx这个命令，判断是0还是1，来实现分布式锁功能，找到一个这样的一个依据点。  

操作步骤： 

​                 a.  setnx命令判断key是否存在，不存在进行操作，存在等待。

​                 b.  expire命令设置一个超时时间，超过这个时间自动释放，避免死锁。

​                 c.  删除这个key，让其他客户端来使用。



#### java代码实现分布式锁 



```java
@Service
public class LockService {

    @Autowired
	private StringRedisTemplate redisTemplate;

	private String redisKey = "lockKey";

    /***
     * 加锁
     * @param waitTime
     * @param timeOut
     * @return
     */
	public String lock(int waitTime, int timeOut) {
		try {
			String uuid = UUID.randomUUID().toString();
			Long endTime = System.currentTimeMillis() + waitTime;
			// 等待几秒获取锁
			while (System.currentTimeMillis() < endTime) {
			    // setnx命令 对应的就是1，不存在
				if (redisTemplate.opsForValue().setIfAbsent(redisKey, uuid, timeOut, TimeUnit.MILLISECONDS) == true)
				{
				    // 加锁成功
				    return uuid;
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
	}

    /***
     * 释放锁
     * @param uuid
     */
    public void unlock(String uuid)
    {
        try {
             // 如果uuid相等，才表示是同一把锁
            if (redisTemplate.opsForValue().get(redisKey).equals(uuid))
            {
                System.out.println(" INFO:释放锁.....  "+Thread.currentThread().getName()+",UUID:"+uuid);
                redisTemplate.delete(redisKey);
            }
        }catch (Exception e){
           e.printStackTrace();
        }
    }
}

```



```java
public class RedisThread extends Thread{

    private int number;

    private LockService lockService;

    public RedisThread(int number,LockService lockService) {
        this.number = number;
        this.lockService = lockService;
    }

    @Override
    public void run() {
        super.run();
        String uuid = lockService.lock(5000,5000);
        if (uuid == null)
        {
            System.out.println(" ERROR: 线程序号:    "+number+","+Thread.currentThread().getName()+",获取锁失败，因获取锁超时");
        }else{
            try {
                System.out.println(" SUCCESS: 线程序号:    "+number+","+Thread.currentThread().getName()+",获取锁成功，锁ID："+uuid+",执行自己业务");
                Thread.sleep(150);

            }catch (Exception e){
               e.printStackTrace();
            }
            lockService.unlock(uuid);
        }
    }
}

```



```java
@RestController
public class RedisController {

	@Autowired
	private StringRedisTemplate redisTemplate;

	@Autowired
    private LockService lockService;

 
    @RequestMapping("/lock")
    public void lock() {
        for (int i = 0; i<30;i++)
        {
            new RedisThread(i,lockService).start();
        }
    }
}
```

运行结果如下

```java
SUCCESS: 线程序号:17, Thread-95,获取锁成功，锁ID：1758e037-5ea2-459b-bfca-293ffba2969d,执行自己业务
INFO:释放锁.....  Thread-95,UUID:1758e037-5ea2-459b-bfca-293ffba2969d
SUCCESS: 线程序号: 2, Thread-80,获取锁成功，锁ID：5976e396-fee2-413a-93ed-f4802b095683,执行自己业务
INFO:释放锁.....  Thread-80,UUID:5976e396-fee2-413a-93ed-f4802b095683
SUCCESS: 线程序号: 3, Thread-81,获取锁成功，锁ID：3ce6d78f-1ecd-4cd8-83fd-081ed66763a0,执行自己业务
INFO:释放锁.....  Thread-81,UUID:3ce6d78f-1ecd-4cd8-83fd-081ed66763a0
SUCCESS: 线程序号:11, Thread-89,获取锁成功，锁ID：f233740a-0762-4874-ab60-42d542e3d616,执行自己业务
INFO:释放锁.....  Thread-89,UUID:f233740a-0762-4874-ab60-42d542e3d616
SUCCESS: 线程序号:12, Thread-90,获取锁成功，锁ID：2937f358-bec9-40f9-bbd3-1a42633f9029,执行自己业务
 ERROR: 线程序号:    1,Thread-79,获取锁失败，因获取锁超时
 ERROR: 线程序号:    4,Thread-82,获取锁失败，因获取锁超时
 ERROR: 线程序号:    9,Thread-87,获取锁失败，因获取锁超时
 ERROR: 线程序号:    19,Thread-97,获取锁失败，因获取锁超时
 ERROR: 线程序号:    16,Thread-94,获取锁失败，因获取锁超时
 ERROR: 线程序号:    24,Thread-102,获取锁失败，因获取锁超时
 ERROR: 线程序号:    23,Thread-101,获取锁失败，因获取锁超时
 ERROR: 线程序号:    29,Thread-107,获取锁失败，因获取锁超时
 ERROR: 线程序号:    28,Thread-106,获取锁失败，因获取锁超时
 INFO:释放锁.....  Thread-90,UUID:2937f358-bec9-40f9-bbd3-1a42633f9029
```


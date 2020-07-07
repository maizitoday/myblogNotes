---
title:       "rocketmq系列一基础认识"
subtitle:    ""
description: "队列的好处，缺点，集群搭建，事物消息，生产者和消费者，架构认识, 幂等性"
date:        2019-03-04
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/istio-install_and_example/post-bg.jpg"
tags:        ["rocketmq","docker安装","消息队列","rocketmq"]
categories:  ["Tech" ]
---

[TOC]

**视频资料：https://www.bilibili.com/video/av59635085?from=search&seid=15914016610413035041**

**好的资料：http://www.tianshouzhi.com/api/tutorials/rocketmq/50**

### 消息队列好处

##### 1. 解耦:   

​      系统和系统之间，通过一个中间层的来进行协调，其他系统的调用就只要订阅这个MQ就可以进行业务处理。如果需要进行加入其他业务，只要订阅这个消息就可以了。 对于上游的业务层，根本不需要修改，他只和MQ对接，这样的话，和其他的业务系统进行了解耦。 

##### 2. 异步

​      对于我们以前的业务处理， 一些不重要的，不影响核心业务的处理也都是串行处理的，如果我们用MQ的话，对于一些不核心的业务可以订阅MQ，异步来出来，节省响应时间。 

##### 3. 削峰（限流）

​    对于一些高峰期的大量用户进入的时候，我们系统可能支持不住，如果我们在前面用MQ把这些大量的请求放入队列中，等高峰期过去了， 然后我们在从队列中慢慢的处理堆积在MQ的请求，这样网站不至于崩溃了。



### TPS

TPS包括一条消息入和一条消息出，加上一次用户数据库访问。（业务TPS = CAPS × 每个呼叫平均TPS）

TPS是软件测试结果的测量单位。一个事务是指一个[客户机](https://www.baidu.com/s?wd=%E5%AE%A2%E6%88%B7%E6%9C%BA&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)向服务器发送请求然后服务器做出反应的过程。客户机在发送请求时开始计时，收到服务器响应后结束计时，以此来计算使用的时间和完成的事务个数。

### QPS

QPS：每秒查询率QPS是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准，在因特网上，作为域名系统服务器的机器的性能经常用每秒查询率来衡量。对应fetches/sec，即每秒的响应请求数，也即是最大吞吐能力。



### 消息队列缺点 

##### 1. 系统可用性降低

系统引入的外部依赖越多，越容易挂掉。本来你就是 A 系统调用 BCD 三个系统的接口就好了，人 ABCD 四个系统好好的，没啥问题，你偏加个 MQ 进来，万一 MQ 挂了咋整，MQ 一挂，整套系统崩溃的，你不就完了？如何保证消息队列的高可用。

##### 2. 系统复杂度提高

硬生生加个 MQ 进来，你怎么[保证消息没有重复消费]？怎么[处理消息丢失的情况]？怎么保证消息传递的顺序性？头大头大，问题一大堆，痛苦不已。

##### 3. 一致性问题

A 系统处理完了直接返回成功了，人都以为你这个请求就成功了；但是问题是，要是 BCD 三个系统那里，BD 两个系统写库成功了，结果 C 系统写库失败了，咋整？你这数据就不一致了。



### RocketMQ

##### 1. 介绍

​    它是纯Java开发，具有高吞吐量、高可用性、适合大规模分布式系统应用的特点。 RocketMQ目前在阿里集团被广泛应用于交易、充值、流计算、消息推送、日志流式处理、binglog分发等场景。

##### 2. 优势

​      a.   面向分布式系统的MQ

​      b.   强调集群无单点，可扩展，任意一点高可用，水平扩展

​      c.   海量消息堆积能力，消息堆积后，写入低延迟

​     d.    支持上万队列

​     e.    消息失败重试机制

​     f.     消息可查询  

​     

#### 专业术语 

##### Producer

​      消息生产者，负责产生消息，一般由业务系统负责产生消息。

##### Consumer

​     消息消费者，负责消费消息，一般是后台系统负责异步消费。

##### Push Consumer（我们一般用这种）

​     Consumer 的一种，应用通常吐 Consumer 对象注册一个 Listener 接口，一旦收到消息，Consumer 对象立 刻回调 Listener 接口方法。

##### Producer  Group

​    一类 Producer 的集合名称，返类 Producer 通常収送一类消息，丏収送逡辑一致。

##### Consumer Group

   一类 Consumer 的集合名称，返类 Consumer 通常消费一类消息，丏消费逡辑一致。

##### Broker

   消息中转角色，负责存储消息，转収消息，一般也称为 Server。在 JMS 规范中称为 Provider。

##### 广播消费

   一条消息被多个 Consumer 消费，即使返些 Consumer 属亍同一个 Consumer Group，消息也会被 Consumer Group 中的每个 Consumer 都消费一次，广播消费中的 Consumer Group 概念可以讣为在消息划分方面无意 丿。

##### 集群消费

一个 Consumer Group 中的 Consumer 实例平均分摊消费消息。例如某个 Topic 有 9 条消息，其中一个 Consumer Group 有 3 个实例（可能是 3 个迕程，戒者 3 台机器），那举每个实例只消费其中的 3 条消息。

##### RocketMQ消息是已文件记录形式持久化的。  

##### 队列集合称为Topic



#### RocketMQ  Producer集群物理部署结构



![屏幕快照 2019-03-13 上午12.23.17](/img/4-9-6.png)

#### ![屏幕快照 2019-03-13 上午12.24.44](/img/4-9-7.png)

Topic可以理解为在rocketMq体系当中作为一个逻辑消息组织形式，一般情况下一类业务消息会申请一个topic来实现业务之间隔离。

Topic是一个逻辑上的概念，实际上在每个broker上以queue的形式保存，也就是说每个topic在broker上会划分成几个逻辑队列，每个逻辑队列保存一部分消息数据，但是保存的消息数据实际上不是真正的消息数据，而是指向commit log的消息索引。



#### Broker集群搭建

推荐的几种 Broker 集群部署方式，返里的 Slave 不可写，但可读，类似亍 Mysql 主备方式。下面是常用的推荐模式

##### 多Master多Slave模式，异步复制

​                主从节点数据同步方式为：异步复制 ，相当于，发送消息过来了，先到master，然后马上返回给客户端，同时，这个时候起一条线程，把数据复制到slave中。 这样，这种模式就是有可能在复制到slave的时候，消息丢失了。

##### 多Master多Slave模式，同步双写

​               主从节点数据同步方式为：异步复制 ，相当于，发送消息过来了，先到master，然后数据复制到slave中，最后在返回给producer，这样的话，消息不会丢失，但是性能会有减少。

以上 Broker 不 Slave 配对是通过挃定相同的 brokerName 参数来配对，Master 的 BrokerId 必须是 0，Slave 的 BrokerId 必须是大亍 0 的数。另外一个 Master 下面可以挂载多个 Slave，同一 Master 下的多个 Slave 通过挃 定丌同的 BrokerId 来区分。



#### 搭建单Master模式

需要注意的是默认的name server 启动是内存4G， broker是默认8G，我们需要修改runserver.sh和runbroker.sh中的JAVA_OPT，修改小一点如：JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m"，然后就可以启动成功了， 不然启动后电脑太卡。 

##### 停止name server 和 broker

```shell
> sh bin/mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

> sh bin/mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
```



##### 启动name server，修改默认端口9876

在/opt/rocketmq/rocketmq-all-4.4.0-bin-release/conf中创建namesrv.properties文件， 加入listenPort=8876，然后下面直接-c 指定这个配置文件启动，启动后，server的端口就是你设置的，如：8876

```shell
nohup sh  bin/mqnamesrv  -c  conf/namesrv.properties > ~/logs/rocketmqlogs/namesrv.log 2>&1

nohup sh  bin/mqnamesrv  -c  conf/namesrv.properties &
```



##### broker-a.properties属性说明

```properties
# 所属集群名字
brokerClusterName=rocketmq-cluster
# 连接外网地址， 如果不设置，会默认用你虚拟机或者本地机子ip 
brokerIP1=192.168.70.102
# ~此处变化 broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
# 0 表示Master, > 0 表示slave
brokerId=0
# `此处变化（此处nameserver跟host配置相匹配，9876为默认rk服务默认端口）nameServer 地址，分号分割
namesrvAddr=192.168.70.102:9876
# Broker 对外服务的监听端口
listenPort=10911
# 删除文件时间点，默认是凌晨4点
deleteWhen=04
# 文件保留时间，默认48小时
fileReservedTime=120
# Broker 的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER 同步双写Master
# - SLAVE
brokerRole=ASYNC_MASTER
# 刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
# #是否允许 Broker 自动创建Topic,建议线下开启,线上关闭 (这个在模拟tools工具发送消息时候要打开，不然阻塞)
autoCreateTopicEnable=true


#######################下面属性后面再进行配置查看效果，先记录#####################



#在发送消息时,自动创建服务器不存在的topic,默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic,建议线下开启,线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组,建议线下开启,线上关闭
autoCreateSubscriptionGroup=true
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条,根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88

#这里是我的 日志配置
#存储路径
storePathRootDir=/usr/local/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/usr/local/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort

#限制的消息大小  
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```



## RocketMQ幂等性问题

转载地址：https://www.cnblogs.com/wishsaber/p/12326154.html

### 什么是幂等性：

在编程中一个幂等操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同。

当出现消费者对某条消息重复消费的情况时，重复消费的结果与消费一次的结果是相同的，并且多次消费并未对业务系统产生任何负面影响，那么这整个过程就实现可消息幂等。

例如，在支付场景下，消费者消费扣款消息，对一笔订单执行扣款操作，扣款金额为 100 元。

如果因网络不稳定等原因导致扣款消息重复投递，消费者重复消费了该扣款消息，但最终的业务结果是只扣款一次，扣费 100 元，

且用户的扣款记录中对应的订单只有一条扣款流水，不会多次扣除费用。那么这次扣款操作是符合要求的，整个消费过程实现了消费幂等。



### 实例：

在RocketMQ中因为 Message ID 有可能出现冲突（重复）的情况，所以真正安全的幂等处理，不建议以 Message ID 作为处理依据。

最好的方式是以业务唯一标识作为幂等处理的关键依据，而业务的唯一标识可以通过消息 Key 设置。

下面模拟：

##### 生产者

```java
public class RetryProducer {
    public static void main(String[] args) throws MQClientException {
        DefaultMQProducer producer = new DefaultMQProducer("retry_rmq-group");
        producer.setNamesrvAddr("192.168.152.55:9876;192.168.152.66:9876");
        producer.setInstanceName("retry_producer");
        producer.start();
        try {
            for (int i = 0; i < 1; i++) {
                Thread.sleep(1000); // 每秒发送一次MQ
                Message msg = new Message("itmayiedu-topic", // topic 主题名称
                        "TagA", // tag 临时值
                        ("retry_itmayiedu-6" + i).getBytes()// body 内容
                );
                msg.setKeys(System.currentTimeMillis() + "");
                SendResult sendResult = producer.send(msg);
                System.out.println(sendResult.toString());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        producer.shutdown();
    }
 
}
```



##### 消费者

```java
public class RetryConsumer {
    static private Map<String, String> logMap = new HashMap<>();
 
    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("retry_rmq-group");
 
        consumer.setNamesrvAddr("192.168.152.55:9876;192.168.152.66:9876");
        consumer.setInstanceName("retry_consumer");
        consumer.subscribe("retry_itmayiedu-topic", "TagA");
 
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                String key = null;
                String msgId = null;
                try {
                    for (MessageExt msg : msgs) {
                        key = msg.getKeys();
                        if (logMap.containsKey(key)) {
                            // 无需继续重试。
                            System.out.println("key:"+key+",无需重试...");
                            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
                        }
                        msgId = msg.getMsgId();
                        System.out.println("key:" + key + ",msgid:" + msgId + "---" + new String(msg.getBody()));
                        int i = 1 / 0;
                    }
 
                } catch (Exception e) {
                    e.printStackTrace();
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;
                } finally {
                    logMap.put(key, msgId);
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
        System.out.println("RetryConsumer Started.");
    }
}

```





### 事务消息

![Xnip2019-09-30_00-42-26](/img/Xnip2019-09-30_00-42-26.png)

**注意**：如果本地事务， 也就是我们的service层，执行时间过长，会执行会查本地事务状态，根据状态来对这个消息进行撤销或者继续进行让消费端进行可见。 

大致可以理解， 如果你需要做有关事物的操作的时候， 先进行你的本地事务操作， 只有在本地事务操作成功的时候，你的这个消息才可以被其他的消费者接受到。 他只会保证生产者这边的事务是OK的， 同时也保证这个消息是一定会发给消费端的。 但是消费端那边的事务是否正确他不能保证，如果出错的话， 需要手工进行干预处理。 



[http://silence.work/2018/08/22/RocketMQ-4-3%E4%BA%8B%E5%8A%A1%E4%BD%BF%E7%94%A8%E4%B8%8E%E5%88%86%E6%9E%90/](http://silence.work/2018/08/22/RocketMQ-4-3事务使用与分析/)

[http://silence.work/2019/05/04/RocketMQ%20Reliablity/](http://silence.work/2019/05/04/RocketMQ Reliablity/)

https://www.infoq.cn/article/2018/08/rocketmq-4.3-release

https://www.cnblogs.com/hzmark/p/rocket_txn.html

https://www.cnblogs.com/hzmark/p/rocket_txn.html



# 跟我学RocketMQ之消息幂等

转载：https://cloud.tencent.com/developer/article/1483781

好文， 写的很透彻。 
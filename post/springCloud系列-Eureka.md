---
title:       "springCloud系列-Eureka"
subtitle:    ""
description: ""
date:        2019-04-17
author:      "麦子"
image:       "https://zhaohuabing.com/img/2018-12-27-the-obstacles-to-put-istio-into-production/background.jpg"
tags:        ["springCloud", "Eureka","服务注册和发现"]
categories:  ["Tech" ]
---

[TOC]

**说明： 本学习主要对尚硅谷周阳大神《SpringCloud视频》记录，一下文字来源这个视频PPT**

**说明： 版本springboot2.14**  **<spring-cloud.version>Greenwich.SR1</spring-cloud.version>**

# Eureka是什么

Eureka 采用了 C-S 的设计架构。Eureka Server 作为服务注册功能的服务器，它是服务注册中心。

而系统中的其他微服务，使用 Eureka 的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。SpringCloud 的一些其他模块（比如Zuul）就可以通过 Eureka Server 来发现系统中的其他微服务，并执行相关的逻辑。

![18-1](/img/18-1.png)

Service Provider服务提供方将自身服务注册到Eureka，从而使服务消费方能够找到。

Service Consumer服务消费方从Eureka获取注册服务列表，从而能够消费服务。

**Eureka包含两个组件：Eureka Server和Eureka Client**

Eureka Server提供服务注册服务

各个节点启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到

EurekaClient是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）

# 搭建Eureka Server集群

通过IDEA直接创建eureka server，需要注意其中他的相关包无法下载的时候，先maven clear，然后删除maven库里面的已经没下载好的包， 然后点击maven projects,  执行reimport，从新建立jar包依赖。

启动类添加：@EnableEurekaServer  注解

application.yml配置如下：

```yml
server:
  port: 5001
eureka:
  instance:
    hostname: eureka5001.com #eureka服务端的实例名称, 我这边做了host映射
  client:  #声明自己是服务端
    register-with-eureka: false  #false表示不向注册中心注册自己。
    fetch-registry: false
    service-url:
     defaultZone: http://eureka5002.com:5002/eureka/,http://eureka5003.com:5003/eureka/
#其他两个服务，把对应的地址和端口修改就OK了。    
```

集群效果如下：

![18-2](/img/18-2.png)

# 搭建Eureka Client客户端

注册相关微服务到cureka server端，

启动类添加：@EnableEurekaClient 注解

 pom.xml配置如下

```xml
 <!--
     添加客户端eureka-client包:
     
     因为我的cloud版本是 <spring-cloud.version>Greenwich.SR1</spring-cloud.version>，所以
     我的eureka-client版本是<version>2.1.1.RELEASE</version>
  -->
<dependencies>
     <dependency>
			  <groupId>org.springframework.cloud</groupId>
			  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>
</dependencies>

```

application.yml如下：

```yml
server:
  port: 8001

spring:
  application:
    name: provide-1 #入驻eureka的服务名称

eureka:
  client:
    service-url:
      defaultZone: http://eureka5001.com:5001/eureka,
                   http://eureka5002.com:5002/eureka,
                   http://eureka5003.com:5003/eureka
```

效果如上图,显示除了Eureka里面注册的服务是provide-1，同时也对应出来ip地址。

# 自我保护机制

什么是自我保护模式？

默认情况下，如果EurekaServer在一定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例（默认90秒）。但是当网络分区故障发生时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题——当EurekaServer节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。一旦进入该模式，EurekaServer就会保护服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）。当网络故障恢复后，该Eureka Server节点会自动退出自我保护模式。

在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。当它收到的心跳数重新恢复到阈值以上时，该Eureka Server节点就会自动退出自我保护模式。它的设计哲学就是宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例。一句话讲解：好死不如赖活着

综上，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留），也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。

**一句话：某时刻某一个微服务不可用了，eureka不会立刻清理，依旧会对该微服务的信息进行保存**

# Eureka和Zookeeper比较

**转载地址： https://www.cnblogs.com/Eternally-dream/p/9833464.html**

**传统数据库：ACID**

​     **NOSQL:   CAP**

![18-4](/img/18-4.png)

![18-3](/img/18-3.png)

 ![18-5](/img/18-5.png)

![18-6](/img/18-7.png)
---
title:       "springCloud系列-Hystrix"
subtitle:    ""
description: ""
date:        2019-04-18
author:      "麦子"
image:       "https://zhaohuabing.com/img/2018-12-27-the-obstacles-to-put-istio-into-production/background.jpg"
tags:        ["springCloud", "熔断", "降级", "雪崩"]
categories:  ["Tech" ]
---



[TOC]



**说明： 本学习主要对尚硅谷周阳大神《SpringCloud视频》记录，一下文字来源这个视频PPT**

**说明： 版本springboot2.14**   **<spring-cloud.version>Greenwich.SR1</spring-cloud.version>**

# Hystrix作用

## 分布式系统面临的问题

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败。

## 服务雪崩

多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”.

对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。

## Hystrix是什么

Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

## 服务降级和熔断

![30-1](/img/30-1.png)

![30-2](/img/30-2.png)

# Hystrix服务搭建

 **provide提供方不需要改变，修改的是consumer，调用方来处理。**

## pom.xml

```xml
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>
```



## application.yml

```yml
eureka:
  client:
    register-with-eureka: false #false表示不向注册中心注册自己,这也是和提供者的一个区别，这里只是做消费
    service-url:
      defaultZone: http://eureka5001.com:5001/eureka
feign:
  hystrix:
    enabled: true
```



## 添加@EnableCircuitBreaker注解

```java
@SpringBootApplication
@EnableFeignClients
@EnableEurekaClient // 配置本应用将使用服务注册和服务发现
@EnableCircuitBreaker
public class ConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApplication.class, args);
	}
}
```





## FeignClient代码修改

```java
@Service
@FeignClient(value = "PROVIDE-DATA",fallback = MyHystrixClientFallbackFactory.class)
public interface DataService {

    @RequestMapping(value = "/provide")
    MaiziUser getDataByFeign();
}
```



```java
@Component
public class MyHystrixClientFallbackFactory implements DataService{
    public MaiziUser getDataByFeign() {
         return new MaiziUser().setName("服务已经停止").setAge(11).setSex("男");
    }
}
```


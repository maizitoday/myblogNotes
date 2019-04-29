---
title:       "springCloud系列-Fegin"
subtitle:    ""
description: ""
date:        2019-04-18
author:      "麦子"
image:       "https://zhaohuabing.com/img/2018-12-27-the-obstacles-to-put-istio-into-production/background.jpg"
tags:        ["springCloud", "Fegin","负载均衡"]
categories:  ["Tech" ]
---

[TOC]

**说明： 本学习主要对尚硅谷周阳大神《SpringCloud视频》记录，以下文字来源这个视频PPT**

**说明： 版本springboot2.14**   **<spring-cloud.version>Greenwich.SR1</spring-cloud.version>**

# Fegin是什么

![20-1](/img/20-1.png)

Feign是一种声明式、模板化的HTTP客户端。在Spring Cloud中使用Feign, 我们可以做到使用HTTP请求远程服务时能与 调用本地方法一样的编码体验，开发者完全感知不到这 是远程方法，更感知不到这是个HTTP请求.

# Fegin搭建

Fegin是客户端，他的主要流程是，通过找到Eureka中注册的服务，找到对应的provide提供的方法，直接暴露一个接口，然后Controller调用远程另一个微服务接口，就像是调用本地接口一样，同时他集成了ribbon的负载均衡。一般这样的接口，可以考虑写入到公共的Maven类中，以便其他的微服务调用。

## Consumer端

pomx修改

```xml
   <dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-openfeign</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>
```



## 定义接口

```java
@Service
@FeignClient(value = "PROVIDE-DATA")
public interface DataFeignService {
    @RequestMapping(value = "/provide")
    MaiziUser getDataByFeign();
}
```



## provide端接口查看

这是对外接口，可以看到和上面的Fegin里面接口DataFeignService这个类里面的相关方法是对应的，也就找到对应的方法了。

```java
@RestController
public class ProvideController {

    @Autowired
    private ProvideService service;

    @RequestMapping("provide")
    public MaiziUser provide()
    {
        return service.buildData();
    }
}
```





## 查看当前eureka服务

![20-2](/img/20-2.png)

**启动类，添加 @EnableFeignClients** 



## application.yml文件

```yml
server:
  port: 9001

eureka:
  client:
    register-with-eureka: false #false表示不向注册中心注册自己,这也是和提供者的一个区别，这里只是做消费
    service-url:
      defaultZone: http://eureka5001.com:5001/eureka,http://eureka5002.com:5002/eureka,http://eureka5003.com:5003/eureka

```

controller类直接调用DataFeignService这个服务就OK了。




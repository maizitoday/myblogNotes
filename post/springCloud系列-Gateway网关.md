---

title:       "springCloud系列-gateway网关"
subtitle:    ""
description: ""
date:        2019-05-16
author:      "麦子"
image:       "https://zhaohuabing.com/img/2018-12-27-the-obstacles-to-put-istio-into-production/background.jpg"
tags:        ["springCloud", "gateway", "网关"]
categories:  ["Tech" ]
---

[TOC]

**说明：文章转载 博客地址： http://www.ityouknow.com，作者：纯洁的微笑**

**说明： 版本springboot2.14**   **<spring-cloud.version>Greenwich.SR1</spring-cloud.version>**

# 路由器和猫区别

猫是什么？

猫叫正名叫【调制解调器】, 电脑通过它才能拨号上网



路由器是什么？

路由器作用是可以一个网线使几台电脑可以同时上网,起到一个分配的作用。上面连接猫,下面连接电脑。

# gateway网关作用

API网关是一个服务器，是系统的唯一入口。从面向对象设计的角度看，它与外观模式类似。API网关封装了系统内部架构，为每个客户端提供一个定制的API。**它可能还具有其它职责，如身份验证、监控、负载均衡、缓存、请求分片与管理、静态响应处理。**
API网关方式的核心要点是，所有的客户端和消费端都通过统一的网关接入微服务，在网关层处理所有的非业务功能。通常，网关也是提供REST/HTTP的访问API。服务端通过API-GW注册和管理服务。

![250417-20171207092518113-1200547245](/img/5-16-1.png)



## 为什么用网关

原文地址：https://www.infoq.cn/article/comparing-api-gateway-performances

API 网关出现的原因是微服务架构的出现，不同的微服务一般会有不同的网络地址，而外部客户端可能需要调用多个服务的接口才能完成一个业务需求，如果让客户端直接与各个微服务通信，会有以下的问题：

1. 客户端会多次请求不同的微服务，增加了客户端的复杂性。
2. 存在跨域请求，在一定场景下处理相对复杂。
3. 认证复杂，每个服务都需要独立认证。
4. 难以重构，随着项目的迭代，可能需要重新划分微服务。例如，可能将多个服务合并成一个或者将一个服务拆分成多个。如果客户端直接与微服务通信，那么重构将会很难实施。
5. 某些微服务可能使用了防火墙 / 浏览器不友好的协议，直接访问会有一定的困难。



## API 网关后的优点

**原文地址：https://www.infoq.cn/article/comparing-api-gateway-performances**

- 易于监控。可以在网关收集监控数据并将其推送到外部系统进行分析。
- 易于认证。可以在网关上进行认证，然后再将请求转发到后端的微服务，而无须在每个微服务中进行认证。
- 减少了客户端与各个微服务之间的交互次数。

![b7cb5f47a80e0910dc1c33d62080ea00](/img/5-16-2.png)





## 网关和Nginx的区别

现在感觉，如果在这里在用上Nginx的话， 首先，网关这个服务应该不需要负载均衡了，所以Nginx不会放在网关服务前面，那就只有放入到后面，我们一般用它来做静态动态分离， 如果放到后面，那么Nginx就相当于一个静态服务器了，在springCloud中，Nginx放到后端，不会在来做负载均衡了，因为我可以把其他服务注册到注册中心就好了， 所以如果springCloud中还需要加入Nginx的话， 还没想明白， 后期在对这一块进行修改处理。

# 路由规则



![spring-cloud-gateway3](/img/5-16-3.png)

## 说明

id：我们自定义的路由 ID，保持唯一
uri：目标服务地址
predicates：路由条件，Predicate 接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如：与，或，非）。
filters：过滤规则，本示例暂时没用。



## 基础配置

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route         
        uri: http://www.ityouknow.com  
        predicates:
          - Path=/*
```



## 通过时间来匹配

```yaml
spring:
  cloud:
    gateway:
      routes:
      #通过时间来匹配
      - id: time_route
        uri: http://ityouknow.com
        predicates:
          #请求时间在 2018年1月20日6点6分6秒之后的所有请求都转发到地址http://ityouknow.com
          - After=2018-01-20T06:06:06+08:00[Asia/Shanghai]    
      - id: after_route
        uri: http://ityouknow.com
        predicates:
         #表示在这个时间之前可以进行路由，在这时间之后停止路由，修改完之后重启项目再次访问地
          - Before=2018-01-20T06:06:06+08:00[Asia/Shanghai]   
```



## 通过 Cookie 匹配

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: http://ityouknow.com
        predicates:
           - Cookie=ityouknow, kee.e
```



## 通过 Host 匹配

Host Route Predicate 接收一组参数，一组匹配的域名列表，这个模板是一个 ant 分隔的模板，用.号作为分隔符。它通过参数中的主机地址作为匹配规则。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: http://ityouknow.com
        predicates:
        - Host=**.ityouknow.com
```



## 通过请求方式匹配

通过请求方式匹配   可以通过是 POST、GET、PUT、DELETE 等不同的请求方式来进行路由。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: http://ityouknow.com
        predicates:
        - Method=GET
```



## 通过请求路径匹配

 如果请求路径符合要求，则此路由将匹配，例如：/foo/1 或者 /foo/bar。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: http://ityouknow.com
        predicates:
        - Path=/foo/{segment}
```



## 通过请求 ip

 通过请求 ip 地址进行匹配  

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: http://ityouknow.com
        predicates:
        - RemoteAddr=192.168.1.1/24
```



## 组合使用 

各种 Predicates 同时存在于同一个路由时，请求必须同时满足所有的条件才被这个路由匹配。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_foo_path_headers_to_httpbin
        uri: http://ityouknow.com
        predicates:
        - Host=**.foo.org
        - Path=/headers
        - Method=GET
        - Header=X-Request-Id, \d+
        - Query=foo, ba.
        - Query=baz
        - Cookie=chocolate, ch.p
        - After=2018-01-20T06:06:06+08:00[Asia/Shanghai]
```



# 注册中心

**只要将 Spring Cloud Gateway 注册到服务中心，Spring Cloud Gateway 默认就会代理服务中心的所有服务**

## pom.xml

```yaml
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
			<version>2.1.1.RELEASE</version>
</dependency>
```



## application.yml

```yaml
server:
  port: 6677
  
eureka:
  client:
    service-url:
      defaultZone: http://eureka5001.com:5001/eureka,http://eureka5002.com:5002/eureka,http://eureka5003.com:5003/eureka

logging:
  level:
    org.springframework.cloud.gateway: debug

spring:
  application:
    name: cloud-gateway-eureka
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
```



## eureka服务查看

![屏幕快照 2019-05-15 下午9.37.53](/img/5-16-4.png)

可以看到我们已经把两个provide服务和gateway服务，注册到了eureka中，这个时候，cloud-gateway-eureka会代理服务中心的所有服务。

## 代理服务运行结果

![屏幕快照 2019-05-15 下午9.54.44](/img/5-16-5.png)

![屏幕快照 2019-05-15 下午9.55.00](/img/5-16-6.png)

provide服务这个方法，我返回不同的结果，可以看到，他是一种轮休负载均衡的访问两个服务。

**注意： 我这里在application.yml配置的时候，没有写明uri所以，我在访问的链接是http://localhost:6677/PROVIDE-DATA/provide是这个，如果我们配置了*uri*: lb://PROVIDE-DATA 这样的话，我们的访问地址就是http://localhost:6677/provide**

# 基于 Filter(过滤器) 实现的高级功能

Spring Cloud Gateway 的 Filter 的生命周期不像 Zuul 的那么丰富，它只有两个：“pre” 和 “post”。

- **PRE**： 这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。
- **POST**：这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的 HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。

Spring Cloud Gateway 的 Filter 分为两种：GatewayFilter 与 GlobalFilter。GlobalFilter 会应用到所有的路由上，而 GatewayFilter 将应用到单个路由或者一个分组的路由上。

Spring Cloud Gateway 内置了9种 GlobalFilter，比如 Netty Routing Filter、LoadBalancerClient Filter、Websocket Routing Filter 等，根据名字即可猜测出这些 Filter 的作者，具体大家可以参考官网内容：[Global Filters](http://cloud.spring.io/spring-cloud-gateway/single/spring-cloud-gateway.html#_global_filters)

## 快速上手

修改application.yml,添加规则,AddRequestParameter GatewayFilter 可以在请求中添加指定参数。

```yaml
spring:
  application:
    name: cloud-gateway-eureka
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
        - id: add_request_parameter_route
          uri: lb://PROVIDE-DATA
          filters:
            - AddRequestParameter=foo, bar
          predicates:
            - Method=GET  
```

实现的效果是，在这个规则的请求下面，将会默认有一个参数foo=bar传入provide服务中。

## 效果图

下面是一个消费端调用Feign访问原始的服务效果：

![屏幕快照 2019-05-15 下午10.21.40](/img/5-16-7.png)

下面是通过gateway服务来访问的服务效果：

![屏幕快照 2019-05-15 下午10.22.36](/img/5-16-8.png)

可以看比原来的服务多了传入过来的参数了。



## Filter 的一些常用功能

### StripPrefix Filter

StripPrefix Filter 是一个请求路径截取的功能，我们可以利用这个功能来做特殊业务的转发。如下面， 我们在浏览器端输入http://localhost:6677/name/maizi/provide 实际上他访问的还是http://localhost:6677/provide，StripPrefix这个表示截取路径的个数。

```yaml
spring:
  application:
    name: cloud-gateway-eureka
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
        - id: add_request_parameter_route
          uri: lb://PROVIDE-DATA
          filters:
            - StripPrefix=2
          predicates:
            - Path=/name/**   
```



### PrefixPath Filter

比如：请求/hello，最后转发到目标服务的路径变为/mypath/hello

```yaml
spring:
  application:
    name: cloud-gateway-eureka
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
        - id: add_request_parameter_route
          uri: lb://PROVIDE-DATA
          predicates:
            - Path=/**
          filters:
            - PrefixPath=/mypath
```



### 限速路由器

限速在高并发场景中比较常用的手段之一，可以有效的保障服务的整体稳定性，Spring Cloud Gateway 提供了基于 Redis 的限流方案。所以我们首先需要添加对应的依赖包`spring-boot-starter-data-redis-reactive`

具体查看转载地址学习。



### 熔断路由器

虽然他可以处理熔断，但是感觉在Consumer端融合hystrix，代码更加的清晰，同时业务分成也更好，网关这一块就只是处理官网东西，hystrix这个更加偏向业务，需要在Consumer这里写更加的清晰。 



### 重试路由器

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: retry_test
        uri: lb://spring-cloud-producer
        predicates:
        - Path=/retry
        filters:
        - name: Retry
          args:
            retries: 3
            statuses: BAD_GATEWAY
```

Retry GatewayFilter 通过这四个参数来控制重试机制： retries, statuses, methods, 和 series。

- retries：重试次数，默认值是 3 次
- statuses：HTTP 的状态返回码，取值请参考：`org.springframework.http.HttpStatus`
- methods：指定哪些方法的请求需要进行重试逻辑，默认值是 GET 方法，取值参考：`org.springframework.http.HttpMethod`
- series：一些列的状态码配置，取值参考：`org.springframework.http.HttpStatus.Series`。符合的某段状态码才会进行重试逻辑，默认值是 SERVER_ERROR，值是 5，也就是 5XX(5 开头的状态码)，共有5 个值。






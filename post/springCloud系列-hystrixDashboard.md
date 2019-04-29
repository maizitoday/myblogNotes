---
title:       "springCloud系列-HystrixDashboard"
subtitle:    ""
description: ""
date:        2019-04-18
author:      "麦子"
image:       "https://zhaohuabing.com/img/2018-12-27-the-obstacles-to-put-istio-into-production/background.jpg"
tags:        ["springCloud", "HystrixDashboard", "监控微服务"]
categories:  ["Tech" ]
---

[TOC]

**说明： 本学习主要对尚硅谷周阳大神《SpringCloud视频》记录，一下文字来源这个视频PPT**

**说明： 版本springboot2.14 ** **<spring-cloud.version>Greenwich.SR1</spring-cloud.version>**

# 作用

![40-1](/img/40-1.png)

# 服务搭建

## IDEA直接创建

添加@EnableHystrixDashboard，这个注解， 然后访问http://localhost:9999/hystrix，需要加入后缀hystrix

## consumer端

### pom.xml

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

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>
		
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>

    <!--主要是这个组件包-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>
```



### application.yml

```yaml
server:
  port: 9001

eureka:
  client:
    register-with-eureka: false #false表示不向注册中心注册自己,这也是和提供者的一个区别，这里只是做消费
    service-url:
      defaultZone: http://eureka5001.com:5001/eureka
feign:
  hystrix:
    enabled: true
```



### 代码层

需要加入下面的代码， springboot2.0，才能收集信息。

```java
@Bean
public ServletRegistrationBean getServlet(){
    HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
    ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
    registrationBean.setLoadOnStartup(1);
    registrationBean.addUrlMappings("/actuator/hystrix.stream");
    registrationBean.setName("HystrixMetricsStreamServlet");
    return registrationBean;
}
```



## 效果图

![40-2](/img/40-2.png)
---
title:       "springCloud系列-config"
subtitle:    ""
description: ""
date:        2019-05-16
author:      "麦子"
image:       "https://images.unsplash.com/photo-1533902767940-2f2bb0062336?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=3450&q=80"
tags:        ["springCloud", "配置文件自动更新"]
categories:  ["Tech" ]
---

[TOC]

**说明： 本学习主要对尚硅谷周阳大神《SpringCloud视频》记录，一下文字来源这个视频PPT**

**说明： 版本springboot2.14**   **<spring-cloud.version>Greenwich.SR1</spring-cloud.version>**

# config作用

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为**各个不同微服务**应用的所有环境提供了一个中心化的**外部**配置。 我们现在可以直接在clone在本地的git配置环境中，进行直接修改相关配置，然后不需要重启对应的服务。更加的方便。

1. 集中管理配置文件
2. 不同环境不同配置，动态化配置更新，分环境部署比如dev/test/prod
3. 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件,服务会向配置中心统一拉取配置自己的配置
4. 当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置。

# config流程

![5-16-11](/img/5-16-11.png)



## config-server 和 git 通信

### pom.xml

```xml
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
</dependency>
```



### application.yml

```yaml
spring:
  application:
    name: service-config
  cloud:
    config:
      server:
        git:
          uri: https://github.com/maizitoday/springcloud-config.git  #git 地址 
          search-paths: 
             -projectA #会在github下面的这个文件夹下面去找相关文件。
          default-label： #分支
```



### 测试访问效果

![5-16-12](/img/5-16-12.png)



### 配置的读取规则 

![5-16-13](/img/5-16-13.png)

上面图，显示访问github上面的application.yml的方式。label是分支的名称。可以根据这种方式来读取git上面的数据



## config-client 和 config-server通信 

### pom.xml

```xml
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```



### bootstrap.yml

```yaml
spring:
  cloud:
    config:
      name: application-client  #指定github上面的确定的资源文件名称，没有后缀yml
      profile: dev  #本次访问的资源项目
      label: master  #当前分支 
      uri: http://localhost:8080  #config 服务， 指定从哪一个服务来获取github上面的资源
```



### github上面资源文件路径和application.yml文件

![5-16-14](/img/5-16-14.png)



### 运行效果

![5-16-15](/img/5-16-15.png)

## 

# 自动刷新

当我们修改github本地文件的时候，需要在浏览器端，马上可以看到最新的数据，需要Spring Cloud Bus 加上 MQ 来实现，然后在github上面需要设置一个钩子， 加入相关指令代码， 然后就可以了 。 后期在加上这一块的知识实现。 



## 


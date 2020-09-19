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



```
AutowiredAnnotationBeanPostProcessor   inject  
```

上面类里面的方法就是最终把数据源赋值给实体的。 

基本思路：

1. 重写数据源EnvironmentRepository，获取mongdb里面的数据。

2. 其他的微服务通过Http的请求，获取到mongdb里面的数据， 然后放到各自的数据源中。 

3. 各自的微服务启动的时候， 扫描所有的@value的注解， 然后找到

4. ```java
   @Value("${xxl.job.admin.addresses}")
       private String adminAddresses;
   ```

   找到${xxl.job.admin.addresses} 值， 然后去掉{} 变成 

   ```java
   xxl.job.admin.addresses
   ```

   如此， 就和你的配置文件一样了。 

   ```properties
   xxl.job.admin.addresses=${xxl-job-admin-addresses}
   
   ### xxl-job executor address
   xxl.job.executor.appname=${spring.application.name}
   xxl.job.executor.ip=
   xxl.job.executor.port=${xxl-job-executor-port}
   ```

同时这个时候可以获取到

```java
${xxl-job-admin-addresses} 
```

这个值。 

5. 查看mongdb的数据源数据

6. ```json
   {
   			"key" : "task",
   			"value" : {
   				"xxl-job-executor-port" : "8209",
   				"xxl-job-executor-logpath" : "/home/XX",
   				"xxl-job-accessToken" : "",
   				"xxl-job-admin-addresses" : "http://XX"
   			}
   		}
   ```

   如此可以看到和这边的

   ```
   xxl-job-admin-addresses
   ```

   对应起来，如此就获取到了值。 注意这里和 key是没有关系的， 加载配置文件的时候，都是加载

   ```
   application_dev.yml
   application_dataSource.yml
   ```

   这样来加载的。 

7. 如此获取到了值， 然后在bean类的时候， 因为在扫描所有的@Value的注解的时候，就可以获取到bean类， 所以进行反射机制 ，这个配置文件的值就填充到了这个实体类了。 


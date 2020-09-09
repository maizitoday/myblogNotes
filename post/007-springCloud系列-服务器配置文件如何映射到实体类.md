---
title:       "springCloud系列-服务器配置文件如何映射到实体类"
subtitle:    ""
description: "源码中mongdb里面的数据如何映射到config service实体中"
date:        2020-08-29
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/istio-install_and_example/post-bg.jpg"
tags:        ["springboot", "springCloud"]
categories:  ["Tech" ]
---

[TOC]

```java
AutowiredAnnotationBeanPostProcessor   inject  
```

上面类里面的方法就是最终把数据源赋值给实体的。 

基本思路：

1. 重写数据源EnvironmentRepository，获取mongdb里面的数据。

2. 其他的微服务通过Http的请求，获取到mongdb里面的数据， 然后放到各自的数据源中。 

3.  各自的微服务启动的时候， 扫描所有的@value的注解， 然后找到

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
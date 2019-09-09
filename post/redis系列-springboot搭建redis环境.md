---
title:       "springboot搭建redis环境"
subtitle:    ""
description: ""
date:        2019-09-09
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["redis", "springboot系列", "springboot搭建redis环境"]
categories:  ["Tech" ]
---



官方文档：<https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#why-spring-redis>

# pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
    <scope>compile</scope>
</dependency>  

<!--springboot2.X默认使用lettuce连接池，需要引入commons-pool2-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency> 

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.54</version>
</dependency>
```

# application.yml

```yml
spring:
  redis:
    host: 127.0.0.1
    port: 6379
    lettuce:
      pool:
        max-active: 200 #连接池最大连接数（使用负值表示没有限制）
        max-idle: 20 # 连接池中的最大空闲连接
        min-idle: 5 #连接池中的最小空闲连接
        max-wait: 1000 # 连接池最大阻塞等待时间（使用负值表示没有限制）
```

# SpringApplication

```java
#通过它可以对数据库（redis)进行操作。基本的实现为CrudRepository。类似Mybatis-plus的CRUD快捷
@EnableRedisRepositories
@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

}
```

# MyController

```java
@RestController
public class MyController {
    
    @Autowired
    private StringRedisTemplate redisTemplate;
   
    @GetMapping(value = "showMsg")
    public String showMsg() {
        String val = redisTemplate.opsForValue().get("maizi");
        System.out.println(val);
        return "hello controller";
    }
}
```


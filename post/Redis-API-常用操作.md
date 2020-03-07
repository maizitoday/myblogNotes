---
title:       "Redis-API-常用操作"
subtitle:    ""
description: ""
date:        2020-02-29
author:      "麦子"
image:       "https://c.pxhere.com/images/91/4d/7a1a53a9399ed55ee697daf983d1-1589459.jpg!d"
tags:        ["redis"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://blog.csdn.net/weixin_43835717/article/details/92802040** 

# 来源

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

# StringRedisTemplate操作方法

```java
StringRedisTemplate.opsForValue().* //操作String字符串类型
StringRedisTemplate.delete(key/collection) //根据key/keys删除
StringRedisTemplate.opsForList().*  //操作List类型
StringRedisTemplate.opsForHash().*  //操作Hash类型
StringRedisTemplate.opsForSet().*  //操作set类型
StringRedisTemplate.opsForZSet().*  //操作有序set
```



## opsForValue(操作字符串)

```

```




















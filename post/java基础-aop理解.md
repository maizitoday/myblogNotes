---
title:       "aop理解"
subtitle:    ""
description: "AOP专业术语，自定义一个AOP"
date:        2019-07-12
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["java基础", "springboot系列", "aop"]
categories:  ["Tech" ]
---

[TOC]

# 概述

AOP：面向切面编程，相对于OOP面向对象编程 Spring的AOP的存在目的是为了解耦。AOP可以让一组类共享相同的行为。在OOP中只能继承和实现接口，且类继承只能单继承，阻碍更多行为添加到一组类上，AOP弥补了OOP的不足。

# Aop好处

利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

# Aop专业术语

通知（Advice）：通知定义了切面是什么以及何时使用；里面就是增强方法的逻辑，分为前置增强，后置增强，环绕增强，最终增强，异常增强。

切入点（Pointcut）：切点有助于缩小切面所通知的连接点的范围，一般通过正则表达式或者自定义注解来处理。

切面（Aspect：切面是通知和切点的结合。通知和切点定义了切面的全部内容，添加辅助代码的地方。

连接点（JoinPoint）：连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时，抛出异常时，甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程中，并添加新的行为。

# Springboot实现Aop



## pom.xml

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```



## 自定义注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyLog {
    String fileName();
}
```



## 切面

```java
@Aspect // 记得这个注解
@Component
public class AopTest {

    @Around("@annotation(myLog)") // 注意这个地方自定义的注解， 头字母要小写，不然报错找不到切点
    public Object around(ProceedingJoinPoint joinPoint, MyLog myLog) throws Throwable {
        System.out.println("方法开始时间是:" + new Date());
        Object o = joinPoint.proceed();
        System.out.println("方法结束时间是:" + new Date());
        return o;
    }

}
```



## 测试Controller

```java
@RestController
public class MyAopController {
    
     @RequestMapping("/myaop")
     @MyLog(fileName = "MyAopController")
     public void showMessage() {
         System.out.println("MyAopController 被切入");
     }
}

运行结果

方法开始时间是:Fri Jul 12 14:45:48 CST 2019
MyAopController 被切入
方法结束时间是:Fri Jul 12 14:45:48 CST 2019
```
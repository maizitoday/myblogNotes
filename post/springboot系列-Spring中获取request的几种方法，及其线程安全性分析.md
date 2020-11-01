---
title:       "springboot系列-Spring中获取request的几种方法，及其线程安全性分析"
subtitle:    ""
description: "获取request的4种方式，基类直接注入request,手动调用, 直接获取request"
date:        2020-11-01
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["springboot系列", "获取request的4种方式，基类直接注入request"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://www.cnblogs.com/kismetv/p/8757260.html**

# 概述

在使用Spring MVC开发Web系统时，经常需要在处理请求时使用request对象，比如获取客户端ip地址、请求的url、header中的属性（如cookie、授权信息）、body中的数据等。由于在Spring MVC中，处理请求的Controller、Service等对象都是单例的，因此获取request对象时最需要注意的问题，便是request对象是否是线程安全的：当有大量并发请求时，能否保证不同请求/线程中使用不同的request对象。

这里还有一个问题需要注意：前面所说的“在处理请求时”使用request对象，究竟是在哪里使用呢？考虑到获取request对象的方法有微小的不同，大体可以分为两类：

1)  在Spring的Bean中使用request对象：既包括Controller、Service、Repository等MVC的Bean，也包括了Component等普通的Spring Bean。为了方便说明，后文中Spring中的Bean一律简称为Bean。

2)    在非Bean中使用request对象：如普通的Java对象的方法中使用，或在类的静态方法中使用。

此外，本文讨论是围绕代表请求的request对象展开的，但所用方法同样适用于response对象、InputStream/Reader、OutputStream/ Writer等；其中InputStream/Reader可以读取请求中的数据，OutputStream/ Writer可以向响应写入数据。

# 方法1：Controller中加参数

## 代码示例

这种方法实现最简单，直接上Controller代码：

```java
@Controller
public class TestController {
    @RequestMapping("/test")
    public void test(HttpServletRequest request) throws InterruptedException {
        // 模拟程序执行了一段时间
        Thread.sleep(1000);
    }
}
```

该方法实现的原理是，在Controller方法开始处理请求时，Spring会将request对象赋值到方法参数中。除了request对象，可以通过这种方法获取的参数还有很多，



## 线程安全性

测试结果：线程安全

分析：此时request对象是方法参数，相当于局部变量，毫无疑问是线程安全的。



## 优缺点

这种方法的主要缺点是request对象写起来冗余太多，主要体现在两点：

1)    如果多个controller方法中都需要request对象，那么在每个方法中都需要添加一遍request参数

2)     request对象的获取只能从controller开始，如果使用request对象的地方在函数调用层级比较深的地方，那么整个调用链上的所有方法都需要添加request参数

实际上，在整个请求处理的过程中，request对象是贯穿始终的；也就是说，除了定时器等特殊情况，request对象相当于线程内部的一个全局变量。而该方法，相当于将这个全局变量，传来传去。



# 方法2：自动注入



## 代码示例

```java
@Controller
public class TestController{
     
    @Autowired
    private HttpServletRequest request; //自动注入request
     
    @RequestMapping("/test")
    public void test() throws InterruptedException{
        //模拟程序执行了一段时间
        Thread.sleep(1000);
    }
}
```



## 线程安全性

测试结果：线程安全

分析：在Spring中，Controller的scope是singleton(单例)，也就是说在整个web系统中，只有一个TestController；但是其中注入的request却是线程安全的，原因在于：使用这种方式，当Bean（本例的TestController）初始化时，Spring并没有注入一个request对象，而是注入了一个代理（proxy）；当Bean中需要使用request对象时，通过该代理获取request对象。

代理对象中用到了 ThreadLocal ， 因此request对象也是线程局部变量；这就保证了request对象的线程安全性。



## 优缺点

该方法的主要优点：

1)      注入不局限于Controller中：在方法1中，只能在Controller中加入request参数。而对于方法2，不仅可以在Controller中注入，还可以在任何Bean中注入，包括Service、Repository及普通的Bean。

2)      注入的对象不限于request：除了注入request对象，该方法还可以注入其他scope为request或session的对象，如response对象、session对象等；并保证线程安全。

3)      减少代码冗余：只需要在需要request对象的Bean中注入request对象，便可以在该Bean的各个方法中使用，与方法1相比大大减少了代码冗余。

但是，该方法也会存在代码冗余。考虑这样的场景：web系统中有很多controller，每个controller中都会使用request对象（这种场景实际上非常频繁），这时就需要写很多次注入request的代码；如果还需要注入response，代码就更繁琐了。下面说明自动注入方法的改进方法，并分析其线程安全性及优缺点。

# 方法3：基类中自动注入

## 代码示例

基类代码：

```java
public class BaseController {
    @Autowired
    protected HttpServletRequest request;     
}
```

## 线程安全性

测试结果：线程安全

分析：在理解了方法2的线程安全性的基础上，很容易理解方法3是线程安全的：当创建不同的派生类对象时，基类中的域（这里是注入的request）在不同的派生类对象中会占据不同的内存空间，也就是说将注入request的代码放在基类中对线程安全性没有任何影响；测试结果也证明了这一点。



## 优缺点

与方法2相比，避免了在不同的Controller中重复注入request；但是考虑到java只允许继承一个基类，所以如果Controller需要继承其他类时，该方法便不再好用。

无论是方法2和方法3，都只能在Bean中注入request；如果其他方法（如工具类中static方法）需要使用request对象，则需要在调用这些方法时将request参数传递进去。下面介绍的方法4，则可以直接在诸如工具类中的static方法中使用request对象（当然在各种Bean中也可以使用）。



# 方法4：手动调用

## 代码示例

```java
@Controller
public class TestController {
    @RequestMapping("/test")
    public void test() throws InterruptedException {
        HttpServletRequest request = ((ServletRequestAttributes) (RequestContextHolder.currentRequestAttributes())).getRequest();
        // 模拟程序执行了一段时间
        Thread.sleep(1000);
    }
}
```

## 线程安全性

测试结果：线程安全

分析：该方法与方法2（自动注入）类似，只不过方法2中通过自动注入实现，本方法通过手动方法调用实现。因此本方法也是线程安全的。

## 优缺点

优点：可以在非Bean中直接获取。缺点：如果使用的地方较多，代码非常繁琐；因此可以与其他方法配合使用。

# 总结

综上所述，Controller中加参数（方法1）、自动注入（方法2和方法3）、手动调用（方法4）都是线程安全的，都可以用来获取request对象。如果系统中request对象使用较少，则使用哪种方式均可；如果使用较多，建议使用自动注入（方法2 和方法3）来减少代码冗余。如果需要在非Bean中使用request对象，既可以在上层调用时通过参数传入，也可以直接在方法中通过手动调用（方法4）获得。






















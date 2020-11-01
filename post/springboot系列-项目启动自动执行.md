---
title:       "springboot系列-项目启动自动执行"
subtitle:    ""
description: "sprigboot项目启动自动执行, 初始化自动执行"
date:        2020-10-31
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["springboot系列", "sprigboot项目启动自动执行"]
categories:  ["Tech" ]
---

[TOC]

**转载：https://blog.csdn.net/fyyyr/article/details/86505304**

# 概述

通常，有些操作需要在工程启动时执行，例如某些资源的加载。SpringBoot提供了几种方式来实现该功能：

# @PostConstruct

对于注入到Spring容器中的类，在其成员函数前添加`@PostConstruct`注解，则在执行Spring beans初始化时，就会执行该函数。

**但由于该函数执行时，其他Spring beans可能并未初始化完成，因此在该函数中执行的初始化操作应当不依赖于其他Spring beans**。

```java
@Component
public class Construct {
    @PostConstruct
    public void doConstruct() throws Exception {
        System.out.println("初始化：PostConstruct");
    }
}

```

# CommandLineRunner

`CommandLineRunner`是Spring提供的接口，定义了一个run()方法，用于执行初始化操作。

```java
@Component
public class InitCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("初始化：InitCommandLineRunner");
    }
}

```

`CommandLineRunner`的时机为Spring beans初始化之后，因此`CommandLineRunner`的执行一定是晚于`@PostConstruct`的。
若有多组初始化操作，则每一组操作都要定义一个`CommandLineRunner`派生类并实现run()方法。这些操作的执行顺序使用`@Order(n)`来设置，n为int型数据。

```java
@Component
@Order(99)
public class CommandLineRunnerA implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("初始化：CommandLineRunnerA");
    }
}

@Component
@Order(1)
public class CommandLineRunnerB implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("初始化：CommandLineRunnerB");
    }
}

```

如上，会先执行`CommandLineRunnerB`的run()，再执行`CommandLineRunnerA`的run()。`@Order(n)`中的n较小的会先执行，较大的后执行。n只要是int值即可，无需顺序递增。

# ApplicationRunner

`ApplicationRunner`接口与`CommandLineRunner`接口类似，都需要实现run()方法。二者的区别在于run()方法的参数不同：

```java
@Component
public class InitApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments applicationArguments) throws Exception {
        System.out.println("初始化：InitApplicationRunner");
    }
}

```

`ApplicationRunner`接口的run()参数为ApplicationArguments对象，因此可以获取更多项目相关的内容。
`ApplicationRunner`接口与`CommandLineRunner`接口的调用时机也是相同的，都是Spring beans初始化之后。因此`ApplicationRunner`接口也使用`@Order(n)`来设置执行顺序。














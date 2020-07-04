---
title:       "Jdk1.8新特性-内置函数式接口"
subtitle:    ""
description: ""
date:        2019-07-02
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "Consumer", "Supplier", "Function", "Predicate", "函数式接口"]
categories:  ["Tech" ]
---

[TOC]

# 用处

函数式接口 (Functional Interface) 就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。

函数式接口是为了 lambda 表达式服务，函数式接口的存在是 lambda 表达式出现的前提，lambda 表达式想关于重写了函数式接口中的唯一方法。



# 比较

就是以前的一种接口的实现方法的另一种形式。 

```java
public class App implements jdk18 {
    public static void main(final String[] args) {

        MyInterface myInterface = (a, b) -> {
            return a + b;
        };
        int show = myInterface.show(1, 2);
        System.out.println(show);

        MyInterface a = new MyInterface() {
            @Override
            public int show(int a, int b) {
                return a + b;
            }
        };

    }
}
```



# 内置4大核心函数式接口

我们用的Lambda表达式是需要函数式接口的，所以java1.8已经内置了很多我们需要的函数式接口。



## Consumer<T> 消费型接口

源码里面的抽象方法

```java
   void accept(T t); // Consumer没有返回值(消费者，有输入，无输出)
```

 实例代码

```java
Consumer<Integer>  consumer = (x) -> System.out.println("x:"+x);
consumer.accept(10);
```



## Supplier<T> 供给型接口

源码里面的抽象方法

```java
  T get(); // 不需要接受参数(供应者，有输出无输入)
```

实例代码

```java
Supplier<Integer>  supplier = () -> {
             return 10 * 10;
        };
int count = supplier.get();
System.out.println(count);
```



## Function<T, R> 函数型接口

源码里面的抽象方法

```java
 R apply(T t); // 有输入，有输出
```

实例代码

```java
Function<Integer,Integer> function = (x) -> {
                return x*2;
        };
int count = function.apply(10);
System.out.println(count);
```



## Predicate<T> 断言型接口

源码里面的抽象方法

```java
 boolean test(T t); // 输入一个参数，并返回一个 Boolean值，
```

实例代码

```java
Predicate<String> predicate = (str) -> {
             return str == null ? false : true;
        };
 boolean flag = predicate.test(null);
 System.out.println(flag);
```

# 其他相关子类

![屏幕快照 2019-07-01 下午10.17.31](/img/屏幕快照 2019-07-01 下午10.17.31.png)

**我们需要知道，还有很多内置的函数式接口。**




---
title:       "jdk1.8新特性-接口"
subtitle:    ""
description: ""
date:        2019-06-29
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "接口", "jdk1.8新特性"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：<https://blog.csdn.net/aitangyong/article/details/54134385>**

# 新特性

接口可以有静态方法，默认方法，也就是说接口中有了实现的方法。



# JDK8Interface

```java
public interface JDK8Interface {

    // static修饰符定义静态方法
    static void staticMethod() {
        System.out.println("接口中的静态方法-----2");
    }

    // default修饰符定义默认方法
    // 我们可以直接调用定义好的静态方法和默认方法。 
    default void defaultMethod() {
        defaultMethod1();
        staticMethod();
        System.out.println("接口中的默认方法-----3");
    }

    default void defaultMethod1() {
        System.out.println("接口1-------中的默认方法-------1");
    }

}
```

**注意：我们可以直接调用定义好的静态方法和默认方法。** 

# JDK8InterfaceImpl

```java
/**
 *   静态方法，只能通过接口名调用，不可以通过实现类的类名或者实现类的对象调用。
     default方法，只能通过接口实现类的对象来调用。
 */
public class JDK8InterfaceImpl implements JDK8Interface {

    public static void main(String[] args) {
        JDK8Interface.staticMethod();

        new JDK8InterfaceImpl().defaultMethod();
    }
    
}
```



# 运行结果

```java
public class JDK8InterfaceImpl implements JDK8Interface {

    public static void main(String[] args) {
        JDK8Interface.staticMethod();
        System.out.println("");
        new JDK8InterfaceImpl().defaultMethod();
    }
    
}

接口中的静态方法-----2

接口1-------中的默认方法-------1
接口中的静态方法-----2
接口中的默认方法-----3
```



# AnotherJDK8InterfaceImpl

```java
public class AnotherJDK8InterfaceImpl implements JDK8Interface {

    /***
     *  签名跟接口default方法一致,但是不能再加default修饰符
     */
    @Override
    public void defaultMethod() {
        System.out.println("接口实现类覆盖了接口中的default");
    }

}
```



# 抽象类和接口的区别

**转载：<https://blog.csdn.net/qq_39907763/article/details/79770854>**

现在抽象类和接口就更像了，那么我们来看看现在接口和抽象类的区别：

1. 抽象类只能单继承，接口可以多现实
2. 抽象类中可以用private、protected方法，接口不可以用

# 接口中加入default方法有什么好处

1. 对于一些公有的方法，直接使用默认的方法，就不用在实现类中写重复代码了。
2. 可以对代码零入侵的加入一些新的方法
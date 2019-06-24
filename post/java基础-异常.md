---
title:       "异常"
subtitle:    ""
description: ""
date:         2019-06-21
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "异常"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://blog.csdn.net/weixin_41987553/article/details/82500591**

# 异常分类

Error:这是我们处理不了的异常。

我们要处理的异常有两种：

编译时被检测异常：
            该异常在编译时，如果没有处理(没有抛也没有try)，编译失败。该异常会被eclipse标识，代表这可以被处理。
        运行时异常(编译时不检测)
            该异常的发生说明，我们需要对某些代码进行修正。

# 异常体系

异常体系的根类是:Throwable

Throwable：
        |--Error:             重大的问题，我们处理不了。也不需要编写代码处理。比如说内存溢出。
        |--Exception:             一般性的错误，是需要我们对编写的代码进行处理。
            |--运行期异常:           在运行时出问题，需要修正代码

​            |--编译期异常，就是一般的异常（编译期异常）

# Throwable类的三个重要方法

```java
getMessage():获取异常信息，返回字符串。
toString():获取异常类名和异常信息，返回字符串。
printStackTrace():获取异常类名和异常信息，以及异常出现在程序中的位置。返回值void。
```

# finally的return问题

```java
public class ReturnDemo {
    
   public static int getData() {
       int a = 100;
        try {
           return a + 1;
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            return a;
        }
   }

   public static void main(String[] args) {
       System.out.println(getData()); // 输出100
   }

}
```

覆盖异常错误的返回如下：

```java
public class ReturnDemo {
    
   public static int getData() {
       int a = 100;
        try {
           int b = a/0;
        } catch (Exception e) {
            e.printStackTrace();
           System.out.println("我已经执行了");
            return 10;
        }finally{
            return a+3;
        }
   }

   public static void main(String[] args) {
       System.out.println(getData());
   }

}


java.lang.ArithmeticException: / by zero
	at com.example.basedemo.exceptiondemo.ReturnDemo.getData(ReturnDemo.java:15)
ReturnDemo.java:15
	at com.example.basedemo.exceptiondemo.ReturnDemo.main(ReturnDemo.java:25)
ReturnDemo.java:25
我已经执行了
103
```

# 自定义异常

定义类继承Exception或者RuntimeException，如果希望写一个检查性异常类，则需要继承 Exception 类。如果你想写一个运行时异常类，那么需要继承 RuntimeException 类。

```java
public class MyException extends Exception {

    public MyException(String message) {
        super(message);
    }

    public MyException() {
    }
}
```

```java

public class ReturnDemo {

    public static int getData() throws MyException {
        int a = 100;
        try {
            int b = a / 0;
        } catch (Exception e) {
            throw new MyException("无法和0相除，请修改参数");
        }
        return 0;
    }

    public static void main(String[] args) {
        try {
            getData();
        } catch (MyException e) {
            e.printStackTrace();
            System.out.println(e.getMessage());
        }
   }

}
```

 

```java
com.example.basedemo.exceptiondemo.MyException: 无法和0相除，请修改参数
	at com.example.basedemo.exceptiondemo.ReturnDemo.getData(ReturnDemo.java:17)
ReturnDemo.java:17
	at com.example.basedemo.exceptiondemo.ReturnDemo.main(ReturnDemo.java:24)
ReturnDemo.java:24
无法和0相除，请修改参数
```

# throws和throw的区别

  A：有throws的时候可以没有throw

​        有throw的时候，如果throw抛的异常是Exception体系，那么必须有throws在方法上声明。

  B：throws用于方法的声明上，其后跟的是异常类名，后面可以跟多个异常类，之间用逗号隔开

​        throw用于方法体中，其后跟的是一个异常对象名




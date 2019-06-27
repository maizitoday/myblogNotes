---
title:       "自定义注解"
subtitle:    ""
description: ""
date:        2019-06-24
author:      ""
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "自定义注解"]
categories:  ["Tech" ]
---



[TOC]

# 注解定义

**转载地址：https://www.cnblogs.com/longshiyVip/p/4707519.html**

注解相当于一种标记，在程序中加了注解就等于为程序打上了某种标记，没加，则等于没有某种标记，以后，javac编译器，开发工具和其他程序可以用反射来了解你的类及各种元素上有无何种标记，看你有什么标记，就去干相应的事。标记可以加在包，类，字段，方法，方法的参数以及局部变量上。**在常用的框架中的@controller @service等这样的注解就是通过定义后，然后反射获取对象，然后实例化bean对象的。** 

# Java中提供了四种元注解

**转载地址:https://blog.csdn.net/u013700502/article/details/79729882**

## @Target	

表示注解作用在什么地方，CONSTRUCTOR 声明在构造器、FIELD 域声明、METHOD 方法声明、PACKAGE 包声明、TYPE 类、接口或者enum声明、PARAMETER参数声明、LOCAL_VALABLE局部变量声明

## @Retention	

表示在什么级别保存注解信息，SOURCE注解在编译器编译时丢弃、CLASS注解在编译之后的class文件中存在，但会被VM丢弃、RUNTIME VM将在运行期也保留注解，因此可以用反射读取注解的信息

## @Documented	

将此注解包含在JavaDoc中

## @Inherited	

允许子类继承父类中的注解

# 具体代码实现

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
     String requestMethod();
     String requestAction();
}
```

 

```java
import java.lang.reflect.Method;

public class AnnotationDemo {

    @MyAnnotation(requestMethod = "get", requestAction = "/getData")
    public void test() {

    }

    public static void main(String[] args) {
        try {
            Class<?> cls = Class.forName("com.example.basedemo.annotationdemo.AnnotationDemo");
            System.out.println("处理的对象-->"+cls);

            Method[] methods = cls.getDeclaredMethods();
            for (Method method : methods) {
                //过滤不含自定义注解AnnotationInfo的方法
                boolean isHasAnnotation = method.isAnnotationPresent(MyAnnotation.class);
                if (isHasAnnotation) {
                    method.setAccessible(true);
                    //获取方法上的注解
                    MyAnnotation aInfo = method.getAnnotation(MyAnnotation.class);
                    if (aInfo == null) return;
                    //解析注解上对应的信息
                    String requestMethod = aInfo.requestMethod();
                    System.out.println("requestMethod: " + requestMethod);

                    String requestAction = aInfo.requestAction();
                    System.out.println("requestAction: " + requestAction);
                }
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

**从上面的列子可以猜测， 我们平常在使用一个请求怎么来对应到一个controller中的方法中， 我们在spring运行的时候，会把所有的请求的url和类里面的方法作为一个键值对，我们在浏览器端请求url的时候，就会去找这个url对应的方法，这样就到具体的方法里面了。** 
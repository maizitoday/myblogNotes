---
title:       "Jdk1.8新特性-Lambda表达式"
subtitle:    ""
description: ""
date:        2019-07-02
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "Lambda表达式"]
categories:  ["Tech" ]
---

[TOC]

# Lambda基础语法

java8中引入了新的操作符 "->" 该操作符称为箭头操作符或者lambda操作符，箭头操作符将lambda表达式拆分两部分：

左侧：lambad 表达式的参数列表

右侧：lambad 表达式红所需要执行的功能，即Lambda体。



## 函数式接口支持

**Lambda表达式需要函数式接口的支持。**

函数式接口：接口中只有一个抽象方法的接口，称为函数式接口。jdk1.8有内置了4个主要的函数式接口。可以使用注解@FunctionlInterface修饰可以检查是否是函数式接口。

# lambda使用

**感觉lambda的方式只是对以前接口的实现和内部类的换种方式的写法。**

```java
@FunctionalInterface
public interface MyFun {

    public static final String _123 = "123";

	public String getMsag(String name, String address);

    public static void main(String[] args) {

        // 1.8中匿名内中的变量默认就是final 类型
        String work = "writer";

        System.out.println("-------------------以前的写法---------------------------");
        MyFun myFun2 = new MyFun() {

            @Override
            public String getMsag(String name, String address) {
                return name+":"+address+":"+work;
            }

        };
        System.out.println(myFun2.getMsag("小强", "邵阳"));

        System.out.println("-------------------lambda写法---------------------------");
        /***
         *  1. 如果lambda体只有一句代码，那么这个时候，我们可以省略return和{}, 直接这样就可以了。
         *  2. lambda中的参数不需要指定类型,编译器通过上下文会自己去判断。
         *  3. 如果只有一个参数可以去掉()
         * 
         *  */
        MyFun myFun = (x, y) -> x +":"+ y+":"+work;  

        /**
         *   MyFun myFun = (x, y) -> {
         *        return x +":"+ y;
         *    };
         *  */
        String messag = myFun.getMsag("麦子", "深圳");
        System.out.println(messag);

    }

}
```

# 注意

其实你在使用的时候， 每次使用的时候，其实只有一个抽象方法， 你只要对应这个抽象方法的参数和返回值来写就OK了。
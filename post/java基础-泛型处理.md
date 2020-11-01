---
title:       "泛型"
subtitle:    ""
description: ""
date:        2019-06-21
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "泛型"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://www.imooc.com/article/details/id/22747**

**转载地址：http://www.sohu.com/a/245549100_796914**

# 作用

泛型就是将类型参数化，其在编译时才确定具体的参数。而当我们指定泛型之后，我们去取出数据后就不再需要进行强制类型转换了，这样就减少了发生强制类型转换的风险。

**泛型只在编译阶段有效**

# 泛型本质

```java
ArrayList<String> a = new ArrayList<String>();
ArrayList<Integer> b = new ArrayList<Integer>();
Class c1 = a.getClass();
Class c2 = b.getClass();
System.out.println(c1 == c2); 
```

这道题输出的结果是 true。因为无论对于 ArrayList<String> 还是 ArrayList<Integer>，它们的 Class 类型都是一直的，都是 ArrayList.class。 也就是说：**泛型只存在于编译阶段，而不存在于运行阶段。**在编译后的 class 文件中，是没有泛型这个概念的。

# 泛型原理

**泛型机制的原理就是参数化类型**，也就是说使用E作为泛型机制的形式参数负责占位，当真正构造对象时需要使用真实的数据类型作为实参传递给E这个形参，从而类中的E全部变成了实参类型。

# 泛型类

```java
public class Home<K extends Animal> {

    public String getAnimalType(K k) {

        return k.getType();
    }
}
```

# 泛型方法

```java
   public <T extends Animal> String getAnimalName(T animal) {

        return animal.getType();
    }
```

- public与返回值中间非常重要，可以理解为声明此方法为泛型方法。
- 表明该方法将使用泛型类型T，此时才可以在方法中使用泛型类型T。
- 与泛型类的定义一样，此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型。

**注意：这里T已经限制了传入的参数必须是Animal类型的子类或者是他自己本身。** 

# 上界通配符<? extends T>

```java
public class Home<K extends Animal>

/**
指定泛型的范围是 T 类型或者是T子类，如指定不是前者之一，就报错。这里就限定了必须是Animal的子类或者他自己。

```

# 下界通配符<? super T>

```java
public class Home<K super Animal>

/**
指定泛型的范围必须是 T 的父类。
```

# 泛型如何实现实例化

```java
public abstract class BaseServiceImpl<M extends MyBaseMapper<T>, T extends BaseEntity, D extends T>
        extends ServiceImpl<M, T> {
```

观察可以看到，  我们实例化的是  D 这个实例对象， 他属于 2 的位置处理。

```java
  private D newDTOclass() {
        D dto = null;
        try {
            ParameterizedType ptype = (ParameterizedType) this.getClass().getGenericSuperclass();
            Class<D> clazz = (Class<D>) ptype.getActualTypeArguments()[2];
            dto = (D) clazz.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return dto;
    }
```








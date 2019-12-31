---
title:       "Lombok"
subtitle:    ""
description: ""
date:        2019-06-20
author:      ""
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "lombok"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://www.cnblogs.com/heyonggang/p/8638374.html**

# 作用

Lombok能通过注解的方式，在编译时自动为属性生成构造器、getter/setter、equals、hashcode、toString方法。出现的神奇就是在源码中没有getter和setter方法，但是在编译生成的字节码文件中有getter和setter方法。这样就省去了手动重建这些代码的麻烦，使代码看起来更简洁些。

# pom.xml

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.20</version>
    <scope>provided</scope>
</dependency>
```

# 相关注解

## @Data

@Data注解在类上，会为类的所有属性自动生成setter/getter、equals、canEqual、hashCode、toString方法，如为final属性，则不会为该属性生成setter方法。



## @Getter/@Setter

可以使用@Getter/@Setter注解，此注解在属性上，可以为相应的属性自动生成Getter/Setter方法。

```java
public class User {
    @Getter
    @Setter
    private String  name;
    @Getter
    @Setter
    private int age;
}
```



## @NonNul

该注解用在属性或构造器上，Lombok会生成一个非空的声明，可用于校验参数，能帮助避免空指针。

```java
@Data
public class User {
    @NonNull
    private String  name;
    @NonNull
    private int age;
}
```

![屏幕快照 2019-06-21 上午10.06.11](/img/屏幕快照 2019-06-21 上午10.06.11.png)

可以看到这里会有提示， 必须要进行设置值。这样避免了空指针异常。



## @Cleanup

该注解能帮助我们自动调用close()方法，很大的简化了代码。

```java
public class LombokTest {

    public static void main(String[] args) throws IOException {
        @Cleanup
        InputStream in = new FileInputStream(args[0]);
        @Cleanup
        OutputStream out = new FileOutputStream(args[1]);
        byte[] b = new byte[10000];
        while (true) {
            int r = in.read(b);
            if (r == -1)
                break;
            out.write(b, 0, r);
        }
    }

}
```

如果是我们以前的写法是需要在finally中关闭流的，如下：

```java
 try {
      OutputStream out = new FileOutputStream(args[1]);
      try {
        byte[] b = new byte[10000];
        while (true) {
          int r = in.read(b);
          if (r == -1) break;
          out.write(b, 0, r);
        }
      } finally {
        if (out != null) {
          out.close();
        }
      }
    } finally {
      if (in != null) {
        in.close();
      }
    }
```

相当于默认会去调用close()方法。 





## @EqualsAndHashCode

默认情况下，会使用所有非静态（non-static）和非瞬态（non-transient）属性来生成equals和hasCode，也能通过exclude注解来排除一些属性。

```java
@EqualsAndHashCode(exclude = {"name"})
```



## @ToString

类使用@ToString注解，Lombok会生成一个toString()方法，默认情况下，会输出类名、所有属性（会按照属性定义顺序），用逗号来分割。



## 构造器

@NoArgsConstructor, @RequiredArgsConstructor and @AllArgsConstructor，无参构造器、部分参数构造器、全参构造器。Lombok没法实现多种参数构造器的重载。

**注意：@Data集合了@ToString、@EqualsAndHashCode、@Getter/@Setter、@RequiredArgsConstructor的所有特性，没有全参构造器。**



## @Log4j2

用于打印日志

```java
@Log4j2
public class LombokTest {

    public static void main(String[] args) {

        log.info("我是消息");
    }
}

打印消息如下：
10:37:35.036 [main] INFO com.example.basedemo.lombokdemo.LombokTest - 我是消息
```

# Lombok工作原理分析

链接：https://www.jianshu.com/p/453c379c94bd

运行时能够解析的注解，必须将@Retention设置为RUNTIME，这样就可以通过反射拿到该注解。java.lang.reflect反射包中提供了一个接口AnnotatedElement，该接口定义了获取注解信息的几个方法，Class、Constructor、Field、Method、Package等都实现了该接口，对反射熟悉的朋友应该都会很熟悉这种解析方式。

其实感觉就是在运行时候，解析写了这些注解的类，然后通过放射机制来实现。

# Lombok的优缺点

## 优点

1. 能通过注解的形式自动生成构造器、getter/setter、equals、hashcode、toString等方法，提高了一定的开发效率
2. 让代码变得简洁，不用过多的去关注相应的方法
3. 属性做修改时，也简化了维护为这些属性所生成的getter/setter方法等

## 缺点

1. 不支持多种参数构造器的重载
2. 虽然省去了手动创建getter/setter方法的麻烦，但大大降低了源代码的可读性和完整性，降低了阅读源代码的舒适度

## 



 
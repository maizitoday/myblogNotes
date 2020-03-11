---
title:       "ThreadLocal-设计模式"
subtitle:    ""
description: ""
date:        2019-12-04
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["多线程和高并发", "设计模式"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://blog.csdn.net/pingpangkuangmo/article/details/84654393**

# ThreadLocal设计模式

ThreadLocal设计模式使用的也很频繁，会经常在各大框架找到它们的踪影，如struts2以及最近正在看的SpringAOP等。主要作用。 

1）ThreadLocal所操作的数据是线程间不共享的。它不是用来解决多个线程竞争同一资源的多线程问题。

2）ThreadLocal所操作的数据主要用于线程内共享数据，可以避免同一线程内函数间的传参数问题。

# 线程之间数据隔离

```java
public class ThreadLocalTest {

    public static void main(String[] args) {
        test1();
    }

    private static void test1() {
        Thread t1 = new Thread() {

            @Override
            public void run() {
                System.out.println(this.getName() + "开始值:" + ThreadUtil.getName());
                ThreadUtil.setName("线程0");
                sleepTime(1000 * 1);
                System.out.println(this.getName() + "设定后:" + ThreadUtil.getName());
            }

        };
        Thread t2 = new Thread() {

            @Override
            public void run() {
                sleepTime(1000 * 1);
                System.out.println(this.getName() + "开始值:" + ThreadUtil.getName());
                ThreadUtil.setName("线程1");
                System.out.println(this.getName() + "设定后:" + ThreadUtil.getName());
            }

        };
        t1.start();
        t2.start();
    }

    private static void sleepTime(long millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}

```



```java
public class ThreadUtil {

    private static ThreadLocal<String> nameLocal = new ThreadLocal<String>();

    public static String getName() {
        return nameLocal.get();
    }

    public static void setName(String name) {
        nameLocal.set(name);
    }
}
```



```java
Thread-0开始值:null
Thread-1开始值:null
Thread-1设定后:线程1
Thread-0设定后:线程0
```



ThreadLocal完全可以这样理解，它就是操作线程数据的工具类，哪个线程调用它的get或set方法，它就会操作调用它的线程中的数据。查看源码知晓， 他是依托线程对象的。 

```java
  public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

**注意：ThreadLocal对象里面存储对象ThreadLocalMap依托的是当前线程创建的Map对象，所以线程和线程之间数据是隔离的。**



# 实现线程内共享数据

同理， 因为他的ThreadLocalMap是依托于线程的，那么在同一个线程中创建的Map对象，这样的话，map的值在这个线程里面就是共享的。 

```java
public class ThreadLocalTest2 {

    public static void main(String[] args) {
        test2();
    }

    private static void test2() {
        Thread t1 = new Thread() {

            @Override
            public void run() {
                fun1();
                fun2();
            }

            private void fun2() {
                System.out.println("在函数2中获取到当前线程的name数据为:" + ThreadUtil.getName());
            }

            private void fun1() {
                ThreadUtil.setName("my own name");
                System.out.println("函数1设置当前线程的name数据为:my own name");
            }

        };
        t1.start();
    }
}

```

```java
函数1设置当前线程的name数据为:my own name
在函数2中获取到当前线程的name数据为:my own name
```

这个例子太简单，然而实际情况是某个线程中几十个函数嵌套调用，如果不将线程内共享数据放进该线程的map集合中，而是将共享数据作为参数传递来传递去，将会一片混乱，杂乱无章，有些函数需要这些参数，有些函数不需要。采用ThreadLocal模式，将数据放进线程的map集合中，想要获取时随时可以通过ThreadLocal对象来获取，方便快捷。
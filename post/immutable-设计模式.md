---
title:       "immutable-设计模式"
subtitle:    ""
description: ""
date:        2019-12-04
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["多线程和高并发", "不可变对象", "设计模式"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://blog.csdn.net/liuxiao723846/article/details/48438437**

# immutable Objects

immutable Objects就是那些一旦被创建，它们的状态就不能被改变的Objects，每次对他们的改变都是产生了新的immutable的对象，而mutable Objects就是那些创建后，状态可以被改变的Objects.

举个例子：String和StringBuilder，String是immutable的，每次对于String对象的修改都将产生一个新的String对象，而原来的对象保持不变，而StringBuilder是mutable，因为每次对于它的对象的修改都作用于该对象本身，并没有产生新的对象。

# 使用Immutable类的好处

immutable objects 比传统的mutable对象在多线程应用中更具有优势，它不仅能够保证对象的状态不被改变，而且还可以不使用锁机制就能被其他线程共享。

1）Immutable对象是线程安全的，可以不用被synchronize就在并发环境中共享

2）Immutable对象简化了程序开发，因为它无需使用额外的锁机制就可以在线程间共享

3）Immutable对象提高了程序的性能，因为它减少了synchroinzed的使用

4）Immutable对象是可以被重复使用的，你可以将它们缓存起来重复使用，就像字符串字面量和整型数字一样。你可以使用静态工厂方法来提供类似于valueOf（）这样的方法，它可以从缓存中返回一个已经存在的Immutable对象，而不是重新创建一个。

# 缺点

immutable也有一个缺点就是会制造大量垃圾，由于他们不能被重用而且对于它们的使用就是”用“然后”扔“，字符串就是一个典型的例子，它会创造很多的垃圾，给垃圾收集带来很大的麻烦。当然这只是个极端的例子，合理的使用immutable对象会创造很大的价值。

# 如何在Java中写出Immutable的类

需要遵循以下几个原则：

1）immutable对象的状态在创建之后就不能发生改变，任何对它的改变都应该产生一个新的对象。

2）Immutable类的所有的属性都应该是final的。

3）对象必须被正确的创建，比如：对象引用在对象创建过程中不能泄露(leak)。

4）对象应该是final的，以此来限制子类继承父类，以避免子类改变了父类的immutable特性。

5）如果类中包含mutable类对象，那么返回给客户端的时候，返回该对象的一个拷贝，而不是该对象本身（该条可以归为第一条中的一个特例）

# 代码演示

```java
public class Person {// 允许同时访问的共享资源类
    private final String name;
    private final String address;

    public Person(final String n, final String a) {
        name = n;
        address = a;
    }

    public String getName() {
        return name;
    }

    public String getAddress() {
        return address;
    }

    public String toString() {
        return "[person:name=" + name + ",address=" + address + "]";
    }

    // 多线程访问者
    public class WorkerThread extends Thread {
        private final Person person;

        public WorkerThread(final Person p) {
            person = p;
        }

        public void run() {
            while (true) {
                System.out.println(Thread.currentThread().getName() + " prints " + person);
            }
        }// 线程的访问并不会影响资源的状态，这里仅仅将其打印出来

    }

    public static void main(final String[] args) {
        final Person p = new Person("Alice", "Alaska");
        p.new WorkerThread(p).start();
        p.new WorkerThread(p).start();
        p.new WorkerThread(p).start();
    }
}

运行结果：
Thread-1 prints [person:name=Alice,address=Alaska]
Thread-2 prints [person:name=Alice,address=Alaska]
Thread-2 prints [person:name=Alice,address=Alaska]
Thread-0 prints [person:name=Alice,address=Alaska]
Thread-2 prints [person:name=Alice,address=Alaska]
Thread-1 prints [person:name=Alice,address=Alaska]
Thread-2 prints [person:name=Alice,address=Alaska]
Thread-0 prints [person:name=Alice,address=Alaska]
Thread-2 prints [person:name=Alice,address=Alaska]
```


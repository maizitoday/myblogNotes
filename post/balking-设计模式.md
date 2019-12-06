---
title:       "balking-设计模式"
subtitle:    ""
description: ""
date:        2019-12-04
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["多线程和高并发", "balking", "设计模式"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://blog.csdn.net/huangxun08/article/details/47081515**

# balking简介

当现在不适合这个操作，或是没有必要进行这个操作时就直接放弃这个操作而回去，这个就是Balking模式。例如王某在餐厅吃饭，当王某需要点餐时喊服务员需要点餐。当服务员A和B都注意到了王某点餐的示意，这时服务员B看到服务员A已经去响应了王某的点餐请求，所以服务员B就不会再过去响应王某的点餐请求。

# 代码示例

## Customer

```java
public class Customer {
    private volatile boolean m_calledService = false;// 取值为false表示没有服务请求，否则有服务请求。
    private String m_Name;

    public Customer(String name) {
        this.m_Name = name;
    }

    /**
     * 获得服务的方法,如果警戒条件不成立，那么直接返回，不再执行操作
     */
    public synchronized boolean GetService(String waiterName, String serviceName) {
        if (!m_calledService) {
            System.out.println("no service  need-----------------");
            return false;
        }
        System.out.println(waiterName + "提供服务-- " + serviceName);
        m_calledService = false;
        return true;
    }

    /**
     * 顾客向服务员发出服务请求,当顾客已经是发出服务要求的状态，那么没有必要,执行更改状态的操作，
       直接返回false,  所以，这也是一个Balking Pattern的设计
     *      
     */
    public synchronized boolean CallService(String serviceName) {
        if (m_calledService) {
            System.out.println("not get service-------------------------");
            return false;
        }
        m_calledService = true;
        System.out.println(m_Name + "需要服务 : " + serviceName);
        return true;
    }
}
```

## Waiter

```java
public class Waiter extends Thread {
    private Customer m_Customer;
    private static int i = 0; // 满二十次计数线程

    public Waiter(String name, Customer customer) {
        super(name);
        this.m_Customer = customer;
    }

    /**
     * 服务员随机等待一段时间后为顾客提供服务        
     */
     @Override
    public void run() {
        Object obj = new Object();
        synchronized (obj) {
            while (i < 10) {
                // 这里控制客服那边的发过来的请求和服务员这边的处理是一致。
                if (m_Customer.GetService(super.getName(), "No_" + i)) {
                    i++;
                }
                // 随机等待一段时间
                try {
                    Random rand = new Random();
                    Thread.sleep(rand.nextInt(1_000));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

## CallServiceThread

```java
public class CallServiceThread extends Thread {
    private Customer m_Customer;
    public CallServiceThread(Customer customer) {
        this.m_Customer = customer;
    }
    /**
     * 线程使得顾客发出一百次服务要求
     */
    @Override
    public void run() {
        int i = 0;
        Object obj = new Object();
        synchronized (obj) {
            while (i < 10) {
                if (m_Customer.CallService("No_" + i)) {
                    i++;
                }
                try {
                    Random rand = new Random();
                    Thread.sleep(rand.nextInt(1_000));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

## CallServiceThread

```java
public class CallServiceThread extends Thread {
    private Customer m_Customer;
    public CallServiceThread(Customer customer) {
        this.m_Customer = customer;
    }

    /**
     * 线程使得顾客发出一百次服务要求
     */
    @Override
    public void run() {
        int i = 0;
        Object obj = new Object();
        synchronized (obj) {
            while (i < 10) {
                if (m_Customer.CallService("No_" + i)) {
                    i++;
                }
                try {
                    Random rand = new Random();
                    Thread.sleep(rand.nextInt(1_000));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

## Main

```java
public class Main {
    public static void main(String[] args) {

        Customer customer = new Customer("顾客1");
        CallServiceThread changer = new CallServiceThread(customer);

        Waiter waiter1 = new Waiter("waiter1", customer);
        Waiter waiter2 = new Waiter("waiter2", customer);

        changer.start();
      
        waiter1.start();
        waiter2.start();
    }

}
```

## 运行结果

```java
顾客1需要服务 : No_0
waiter1提供服务-- No_0
no service  need----------------------------------------------
no service  need----------------------------------------------
顾客1需要服务 : No_1
waiter1提供服务-- No_1
no service  need----------------------------------------------
no service  need----------------------------------------------
顾客1需要服务 : No_2
not get service----------------------------------------------
waiter2提供服务-- No_2
no service  need----------------------------------------------
no service  need----------------------------------------------
no service  need----------------------------------------------
顾客1需要服务 : No_3
not get service----------------------------------------------
waiter1提供服务-- No_3
no service  need----------------------------------------------
顾客1需要服务 : No_4
not get service----------------------------------------------
waiter2提供服务-- No_4
顾客1需要服务 : No_5
waiter1提供服务-- No_5
no service  need----------------------------------------------
顾客1需要服务 : No_6
waiter2提供服务-- No_6
顾客1需要服务 : No_7
waiter1提供服务-- No_7
no service  need----------------------------------------------
no service  need----------------------------------------------
顾客1需要服务 : No_8
waiter1提供服务-- No_8
no service  need----------------------------------------------
no service  need----------------------------------------------
no service  need----------------------------------------------
顾客1需要服务 : No_9
waiter2提供服务-- No_9
```

# 总结

上面的示例可以看出来， 一个客服对应两个服务员， 两个服务员的两个线程，通过一个变量

```java
 private volatile boolean m_calledService = false;
```

来控制，已经服务过或者没有服务两个状态的控制来处理， 如果这个服务已经处理过了，那就改变状态了。另外一个状态查看这个状态后，就知道已经服务过了，所以就直接return了。

**这个模式的主要就是变量的控制。**
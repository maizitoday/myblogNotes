---
title:       "Count-Down-设计模式"
subtitle:    ""
description: ""
date:        2019-12-06
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["多线程和高并发", "阀门模式", "设计模式"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://blog.csdn.net/weixin_40288381/article/details/87474970**

# 概述

Count-Down设计模式其实也叫做Latch(阀门)设计模式。当若干个线程并发执行完某个特定的任务，然后等到所有的子任务都执行结束之后再统一汇总。

# 示例代码

```java
public class Latch {
    // 阀门值
    private final int latchnum;

    // 计数器
    private int count = 0;

    public Latch(int latchnum) {
        this.latchnum = latchnum;
    }

    public synchronized void countDown() {
        this.count++;
        System.out.println("countDown ---> " + Thread.currentThread().getName());
        if (this.count == this.latchnum) {
            this.notifyAll();
        }

    }

    public synchronized void await() {
        System.out.println("wait " + Thread.currentThread().getName());
        try {
            this.wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Latch latch = new Latch(5);
        System.out.println("线程第一阶段开始工作");

        for (int i = 0; i < 5; i++) {
            new Thread() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1000);
                        latch.countDown();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }.start();
        }
        latch.await();
        System.out.println("阶段一全部完成，第二阶段开工");
    }
}
```

```java
线程第一阶段开始工作
wait main
countDown ---> Thread-3
countDown ---> Thread-4
countDown ---> Thread-0
countDown ---> Thread-2
countDown ---> Thread-1
阶段一全部完成，第二阶段开工
```



# 总结

可以看到， 我们先对Latch这个进行了加锁，然后main在处理的时候直接wait， 然后其他的线程只有在规定的条件

```java
if (this.count == this.latchnum) {
            this.notifyAll();
    }
```

时候，才进行唤醒锁，也就是等其他的线程都已经完成了。 我们才在最后进行处理。 
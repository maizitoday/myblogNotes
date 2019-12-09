---
title:       "Work-Thread-设计模式"
subtitle:    ""
description: ""
date:        2019-12-06
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["多线程和高并发", "Work-Thread", "设计模式"]
categories:  ["Tech" ]
---

[TOC]

**转载：https://blog.csdn.net/weixin_33997389/article/details/88809045**

# 概述

Work Thread模式和Thread-Per-Message模式类似，Thread-Per-Message每次都创建一个新的线程处理请求，而Work Thread模式预先会创建一个线程池（Thread Pool），每次从线程池中取出线程处理请求。

# 代码示例

## Request请求类

```java
public class Request {
    private final String name;
    private final int number;
    private static final Random random = new Random();
    public Request(String name, int number) {
        this.name = name;
        this.number = number;
    }
    public void execute() {
        System.out.println(Thread.currentThread().getName() + " executes " + this);
        try {
            Thread.sleep(random.nextInt(1000));
        } catch (InterruptedException e) {
        }
    }
    public String toString() {
        return "[ Request from " + name + " No." + number + " ]";
    }
}


```

## ClientThread

```java
public class ClientThread extends Thread {
    private final Channel channel;
    private static final Random random = new Random();
    public ClientThread(String name, Channel channel) {
        super(name);
        this.channel = channel;
    }
    public void run() {
        try {
            for (int i = 0; true; i++) {
                Request request = new Request(getName(), i);
                channel.putRequest(request);
                Thread.sleep(random.nextInt(1000));
            }
        } catch (InterruptedException e) {
        }
    }
}


```

## WorkerThread

```java
public class WorkerThread extends Thread {
    private final Channel channel;
    public WorkerThread(String name, Channel channel) {
        super(name);
        this.channel = channel;
    }
    public void run() {
        while (true) {
            Request request = channel.takeRequest();
            request.execute();
        }
    }
}
```



## Channel

```java
public class Channel {
    private static final int MAX_REQUEST = 100; // 最大请求数
    private volatile  Queue<Request> requestQueue; 
    private final WorkerThread[] threadPool;

    public Channel(int threads) {
        this.requestQueue = new LinkedList<>();
        threadPool = new WorkerThread[threads];
        for (int i = 0; i < threadPool.length; i++) {
            threadPool[i] = new WorkerThread("Worker-" + i, this);
        }
    }

    public void startWorkers() {
        for (int i = 0; i < threadPool.length; i++) {
            threadPool[i].start();
        }
    }

    public synchronized void putRequest(Request request) {
        requestQueue.offer(request);
        notifyAll();
    }

    public synchronized Request takeRequest() {
        while (null == requestQueue.peek()) {
            try {
                wait();
            } catch (InterruptedException e) {
            }
        }
        Request request = requestQueue.remove();
        notifyAll();
        return request;
    }
}
```

## Main

```java
public class Main {
    public static void main(String[] args) {
        Channel channel = new Channel(5);
        channel.startWorkers();
        new ClientThread("Alice", channel).start();
        new ClientThread("Bobby", channel).start();
        new ClientThread("Chris", channel).start();
    }
}
```

## 运行结果

```java
Worker-2 executes [ Request from Chris No.0 ]
Worker-4 executes [ Request from Alice No.0 ]
Worker-3 executes [ Request from Bobby No.0 ]
Worker-0 executes [ Request from Alice No.1 ]
Worker-0 executes [ Request from Chris No.1 ]
Worker-1 executes [ Request from Chris No.2 ]
Worker-2 executes [ Request from Chris No.3 ]
Worker-3 executes [ Request from Bobby No.1 ]
Worker-1 executes [ Request from Alice No.2 ]
Worker-0 executes [ Request from Chris No.4 ]
Worker-4 executes [ Request from Alice No.3 ]
Worker-0 executes [ Request from Bobby No.2 ]
Worker-4 executes [ Request from Alice No.4 ]
Worker-2 executes [ Request from Bobby No.3 ]
Worker-0 executes [ Request from Bobby No.4 ]
```

## 总结

可以看到， 这里的线程池就是开启几条线程在不停的进行循环， 同时，进行生产者和消费者的模式一个演示处理。 
---
title:       "Guarded-Suspension-设计模式"
subtitle:    ""
description: ""
date:        2019-12-04
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["多线程和高并发", "保护性暂挂模式", "设计模式"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://blog.csdn.net/weixin_42245930/article/details/88761176**

# Guarded Suspension

Guarded 是被守护，被保护的意思，Suspension 是暂停的意思。如果执行现在的处理会造成问题，就让执行处理的线程进行等待。**当现在并不适合马上执行某个操作时，就要求想要执行该操作的线程等待，**这就是Guarded Suspension Pattern。Guarded Suspension Pattern 会要求线程等候，以保障实例的安全性。 **其实和生产者和消费者模式有点类似**。 

# Request

```java
public class Request {
    private final String name;

    public Request(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "Request{" + "name='" + name + '\'' + '}';
    }
}
```

# RequestQueue

```java
public class RequestQueue {

    private final Queue<Request> queue = new LinkedList<>();

    // 取出最先放在RequestQueue的中的一个请求，作为返回值，
    // 如果一个请求也没有就一直等待，直到其他线程执行putRequest.
    public synchronized Request getRequest() {
        while (queue.peek() == null) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        return queue.remove();

    }

    // 添加一个请求
    public synchronized void putRequest(Request request) {
        queue.offer(request);
        notifyAll();
    }
}

```

# ClientThread

```java
public class ClientThread extends Thread {
    private final Random random;
    private final RequestQueue requestQueue;

    public ClientThread(RequestQueue requestQueue, String name, long seed) {
        super(name);
        this.requestQueue = requestQueue;
        this.random = new Random(seed);

    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            Request request = new Request("No." + i);
            System.out.println(Thread.currentThread().getName() + " requests " + request+"/n");
            requestQueue.putRequest(request);
            try {
                // 为了错开发送请求的执行点，使用java.util.Random类随机生成0-1000之间的数
                Thread.sleep(random.nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```

# ServerThread

```java
/**
 * 用于表示接收请求的线程，该类也持有RuquestQueue的实例，ServerThread使用该实例的getRequst方法来接受请求。
 */
public class ServerThread extends Thread {

    private final Random random;
    private final RequestQueue requestQueue;

    public ServerThread(RequestQueue requestQueue, String name, long seed) {
        super(name);
        this.requestQueue = requestQueue;
        this.random = new Random(seed);
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            Request request = requestQueue.getRequest();
            System.out.println(Thread.currentThread().getName() + " handles " + request);
            try {
                Thread.sleep(random.nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }

    }
}
```

# Main

```java
public class Main {
    public static void main(String[] args) {
        RequestQueue requestQueue = new RequestQueue();
        new ClientThread(requestQueue, "发送请求-->", 3141592L).start();
        new ServerThread(requestQueue, "处理请求-->", 6535897L).start();
    }
}
```

# 运行结果

```java
送请求--> requests Request{name='No.0'}
处理请求--> handles Request{name='No.0'}

发送请求--> requests Request{name='No.1'}
发送请求--> requests Request{name='No.2'}

处理请求--> handles Request{name='No.1'}
处理请求--> handles Request{name='No.2'}

发送请求--> requests Request{name='No.3'}
处理请求--> handles Request{name='No.3'}
```


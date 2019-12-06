---
title:       "Thread-Per-Message-设计模式"
subtitle:    ""
description: ""
date:        2019-12-06
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["多线程和高并发", "Thread-Per-Message", "设计模式"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://blog.csdn.net/qq_33366098/article/details/89183926**

# 概述

Thread-Per-Message模式是说为每个请求都分配一个线程，由这个线程来执行处理，使得消息能够并发（但是注意：线程的创建是有限的，可以使用线程池来处理，超过数量则加入等待队列），这里包含两个角色，请求的提交线程和请求的执行线程。

# 代码演示

## Message

```java
public class Message {
    private String message;

    public Message(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
}
```

## MessageHandler

```java
public class MessageHandler {

    private static final ExecutorService pool = Executors.newFixedThreadPool(5);

    public void request(final Message message){
        pool.execute(new Runnable() {
            @Override
            public void run() {
                String result = message.getMessage();
                try {
                    Thread.sleep(100);
                    System.out.println(Thread.currentThread().getName() + ">>>>>" + result+"---->"+pool);
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        });
    }
}
```

## ThreadPerMessageTest

```java
public class ThreadPerMessageTest {
    public static void main(String[] args) {
        MessageHandler messageHandler = new MessageHandler();
        for (int i = 0; i < 5; i++){
            messageHandler.request(new Message("信息：" + i));
        }
    }
}
```

# 总结

Thread-Per-Message模式可以提高程序的响应性，但是并不能保证操作的顺序，而且没有返回值。一般用于简单服务器实现，当服务器主线程收到客户端的请求之后就创建一个新的线程去处理客户端的请求，而服务器主线程则返回继续接收客户端请求。

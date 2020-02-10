---
title:       "NIO的网络编程-多线程(二)"
subtitle:    ""
description: ""
date:        2020-02-10
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["网络编程","java基础"]
categories:  ["Tech" ]
---

[TOC]

# NIO 服务器端

```java
package maizi.nio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Set;

/**
 * NIO 服务器端
 * 
 * @author MOTUI
 * 
 */
public class NIOServerSocket {

    // 存储SelectionKey的队列
    private static List<SelectionKey> writeQueen = new ArrayList<SelectionKey>();
    private static Selector selector = null;

    // 添加SelectionKey到队列
    public static void addWriteQueen(SelectionKey key) {
        synchronized (writeQueen) {
            writeQueen.add(key);
            // 唤醒主线程
            selector.wakeup();
        }
    }

    public static void main(String[] args) throws IOException {

        // 1.创建ServerSocketChannel
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        // 2.绑定端口
        serverSocketChannel.bind(new InetSocketAddress(8989));
        // 3.设置为非阻塞
        serverSocketChannel.configureBlocking(false);
        // 4.创建通道选择器
        selector = Selector.open();

        /*
         * 5.注册事件类型
         * 
         * sel:通道选择器 ops:事件类型 ==>SelectionKey:包装类，包含事件类型和通道本身。四个常量类型表示四种事件类型
         * SelectionKey.OP_ACCEPT 获取报文 SelectionKey.OP_CONNECT 连接 SelectionKey.OP_READ 读
         * SelectionKey.OP_WRITE 写
         */
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        while (true) {
            System.out.println("服务器端：正在监听8989端口");
            // 6.获取可用I/O通道,获得有多少可用的通道
            int num = selector.select();
            if (num > 0) { // 判断是否存在可用的通道
                // 获得所有的keys
                Set<SelectionKey> selectedKeys = selector.selectedKeys();
                // 使用iterator遍历所有的keys
                Iterator<SelectionKey> iterator = selectedKeys.iterator();
                // 迭代遍历当前I/O通道
                while (iterator.hasNext()) {
                    // 获得当前key
                    SelectionKey key = iterator.next();
                    // 调用iterator的remove()方法，并不是移除当前I/O通道，标识当前I/O通道已经处理。
                    iterator.remove();
                    // 判断事件类型，做对应的处理
                    if (key.isAcceptable()) {
                        ServerSocketChannel ssChannel = (ServerSocketChannel) key.channel();
                        SocketChannel socketChannel = ssChannel.accept();

                        System.out.println("处理请求：" + socketChannel.getRemoteAddress());

                        // 获取客户端的数据
                        // 设置非阻塞状态
                        socketChannel.configureBlocking(false);
                        // 注册到selector(通道选择器)
                        socketChannel.register(selector, SelectionKey.OP_READ);

                    } else if (key.isReadable()) {
                        // 取消读事件的监控   注意1号代码
                        key.cancel();
                        // 调用读操作工具类
                        RequestProcessor.ProcessorRequest(key);
                    } else if (key.isWritable()) {
                        // 取消读事件的监控
                        key.cancel();
                        // 调用写操作工具类
                        ResponeProcessor.ProcessorRespone(key);
                    }
                }
            } else {
                synchronized (writeQueen) {
                    while (writeQueen.size() > 0) {
                        SelectionKey key = writeQueen.remove(0);
                        // 注册写事件
                        SocketChannel channel = (SocketChannel) key.channel();
                        Object attachment = key.attachment();
                        channel.register(selector, SelectionKey.OP_WRITE, attachment);
                    }
                }
            }
        }
    }
}
```



# 读操作的工具类

```java
package maizi.nio;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.SocketChannel;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 读操作的工具类
 * 
 * @author MOTUI
 *
 */
public class RequestProcessor {

    // 构造线程池
    private static ExecutorService executorService = Executors.newFixedThreadPool(10);

    public static void ProcessorRequest(final SelectionKey key) {
        // 获得线程并执行
        executorService.submit(new Runnable() {

            @Override
            public void run() {
                try {
                    SocketChannel readChannel = (SocketChannel) key.channel();
                    // I/O读数据操作
                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    ByteArrayOutputStream baos = new ByteArrayOutputStream();
                    int len = 0;
                    while (true) {
                        buffer.clear();
                        len = readChannel.read(buffer);
                        if (len == -1)
                            break;
                        buffer.flip();
                        while (buffer.hasRemaining()) {
                            baos.write(buffer.get());
                        }
                    }
                    System.out.println("服务器端接收到的数据：" + new String(baos.toByteArray()));

                    // 将数据添加到key中
                    key.attach(baos);
                    // 将注册写操作添加到队列中
                    NIOServerSocket.addWriteQueen(key);

                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

# 写操作工具类

```java
package maizi.nio;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.SocketChannel;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 写操作工具类
 * 
 * @author MOTUI
 *
 */
public class ResponeProcessor {
    // 构造线程池
    private static ExecutorService executorService = Executors.newFixedThreadPool(10);

    public static void ProcessorRespone(final SelectionKey key) {
        // 拿到线程并执行
        executorService.submit(new Runnable() {
            @Override
            public void run() {
                try {
                    // 写操作
                    SocketChannel writeChannel = (SocketChannel) key.channel();
                    // 拿到客户端传递的数据
                    ByteArrayOutputStream attachment = (ByteArrayOutputStream) key.attachment();

                    System.out.println("客户端发送来的数据：" + new String(attachment.toByteArray()));

                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    String message = "你好，我好，大家好！！";
                    buffer.put(message.getBytes());
                    buffer.flip();
                    writeChannel.write(buffer);
                    writeChannel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

# NIO 客户端

```java
package maizi.nio;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;

/**
 * NIO 客户端
 */
public class NIOClientSocket {

    public static void main(String[] args) throws IOException {
        // 使用线程模拟用户 并发访问
        for (int i = 0; i < 2; i++) {
            new Thread() {
                public void run() {
                    try {
                        // 1.创建SocketChannel
                        SocketChannel socketChannel = SocketChannel.open();
                        // 2.连接服务器
                        socketChannel.connect(new InetSocketAddress("127.0.0.1", 8989));

                        // 写数据
                        String msg = "我是客户端" + Thread.currentThread().getId();
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        buffer.put(msg.getBytes());
                        buffer.flip();
                        socketChannel.write(buffer);
                        socketChannel.shutdownOutput();

                        // 读数据
                        ByteArrayOutputStream bos = new ByteArrayOutputStream();
                        int len = 0;
                        while (true) {
                            buffer.clear();
                            len = socketChannel.read(buffer);
                            if (len == -1)
                                break;
                            buffer.flip();
                            while (buffer.hasRemaining()) {
                                bos.write(buffer.get());
                            }
                        }

                        System.out.println("客户端收到:" + new String(bos.toByteArray()));

                        socketChannel.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                };
            }.start();
        }
    }
}
```



# 注意

## 注意1号代码-->说明

在主线程中，读操作会执行多次。因为在子线程没有执行结束，主线程认为读操作没有结束（在通道选择器中读操作的管道一直存在），会一直开启新的线程去执行读操作，这样就会产生严重的错误。产生这种问题的原因是我们将读操作交给子线程去处理，**主线程依然对此通道保留监控，所以会一直执行读操作**，我们取消主线程对此通道的监控。将服务器端代码稍作修改.所以需要 添加

```java
key.cancel()
```

# 总结

对于NIO的多线程网络编程：NIO也是使用的阻塞I/O,使用了通道选择器进行选择就绪的通道进行处理I/O，达到非阻塞的目的。

## 编程思路

```java
 1.创建ServerSocketChannel
 2.绑定端口
 3.设置为非阻塞
 4.创建通道选择器
 5.注册事件类型
 6.获取可用I/O通道,获得有多少可用的通道
 7.将耗时的读写操作开启子线程去处理
 8.将注册写操作交给主线程处理
```

**重点是7、8两步，如何将注册操作交给主线程处理。**



## 执行流程

### 1.  启动TestServiceSocketChannel（服务器），加载服务器端中的代码

```java
进入到第一个while（true）{
    停留到int num = selector.select();查询是否有可用通道
}
```



### 2.   启动TestClientSocketChannel（客户端），加载客户端中的代码



### 3.   这时客户端有可用的通道

1. ```java
   开始向下执行进入下个while（true）循环，然后将这个iterator.remove(); 标记移除，以为这个请已经被处理，（防止高并发时进行重复处理）
   ```



### 4.   判断当前时间类型并进行处理

**a：**key.isAcceptable()，key.channeal拿到一个通道，并用通道接收报文请求，然后进行通道设置为非阻塞的，然后再将读操作注入到通道里。执行完毕后，此时跳出当前while（iterator.hasNext()因为此时测试的环境是一个线程，如果是多个线程则进行循环处理

**b：**因为在步骤1中已经将读取所以进行判断后会进入else if(key.isReadable()){这里边，进入后会将key.cancel();(会将这个key做上标记)调用RequestProcess.request(key);这个类中的方法进行读操作。此时，服务器端的主线程会阻塞。因为此时已经没有要处理的通道或者说是key的时间

**c：**进入读操作后，执行代码并将从客户端接受到数据，防止到key的附件中key.attach(bos);然后调用服务器端的TestServiceSocketChannel.addWriteQueen(key);方法，将可以存入WriteQueen集合中，存入后会将已经 阻塞的主线程 进行唤醒（selector.wakeup();） 

**d：**唤醒主线程后，以为此时已经有了所对象，会进行注册写入事件和从客户端读取到的数据 放入通道，进行写入操作.进行判断if(key.isWritable())取消key的写标记，调用写方法ResponseProcess.response(key);将写入数据传到客户端，释放通道，完成响应。

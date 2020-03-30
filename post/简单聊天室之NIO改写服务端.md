---
title:       "NIO群聊"
subtitle:    ""
description: ""
date:        2020-03-20
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["网络编程"]
categories:  ["Tech" ]
---

[TOC]

**说明文章资料来源：尚硅谷韩顺平Netty视频教程（2019发布） https://www.bilibili.com/video/av76227904?p=36 **

# 说明

在前面的《简易聊天室案例》文章的基础上进行修改 **TCPServer类 和 ClientHandler类**。



![Xnip2020-01-30_18-01-38](/img/Xnip2020-01-30_18-01-38.png)



![Xnip2020-01-26_13-47-53](/img/Xnip2020-01-26_13-47-53.png)

# GroupChatServer

```java
 package com.nio.chargroup;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-03-19 15:10:49
 * @LastEditTime: 2020-03-20 13:52:47
 * @LastEditors: 麦子
 */

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.Channel;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;

public class GroupChatServer {

    // 定义属性
    private Selector selector;
    private ServerSocketChannel listenChannel;
    private static final int PORT = 6667;

    // 初始化工作
    public GroupChatServer() {

        try {
            // 得到选择器
            selector = Selector.open();
            listenChannel = ServerSocketChannel.open();
            // 绑定端口
            listenChannel.socket().bind(new InetSocketAddress(PORT));
            // 设置非阻塞模式
            listenChannel.configureBlocking(false);
            // 将该 listenChannel 注册到 selector
            listenChannel.register(selector, SelectionKey.OP_ACCEPT);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 监听
    public void listen() {
        try {
            while (true) {
                int count = selector.select();
                if (count > 0) {// 有事件处理
                    // 遍历得到 selectionKey 集合
                    Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                    while (iterator.hasNext()) {
                        // 取出 selectionkey
                        SelectionKey key = iterator.next();
                        if (key.isAcceptable()) {
                            SocketChannel sc = listenChannel.accept();
                            sc.configureBlocking(false);
                            // 将该 sc 注册到 seletor
                            sc.register(selector, SelectionKey.OP_READ);
                            // 提示
                            System.out.println(sc.getRemoteAddress() + " 上线 ");
                        }
                        if (key.isReadable()) {// 通道发送 read 事件，即通道是可读的状态
                            // 处理读 (专门写方法..)
                            readData(key);
                        }
                        // 当前的 key 删除，防止重复处理
                        iterator.remove();
                    }
                } else {
                    System.out.println("等待....");
                }
            }

        } catch (Exception e) {
            e.printStackTrace();

        } finally { // 发生异常处理....

        }
    }

    // 读取客户端消息
    private void readData(SelectionKey key) {
        // 取到关联的 channle
        SocketChannel channel = null;

        try { 
            // 得到 channel
            channel = (SocketChannel) key.channel(); 
            // 创建 buffer
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            // 根据 count 的值做处理
            int count = channel.read(buffer); 
            if (count > 0) { 
                // 把缓存区的数据转成字符串
                String msg = new String(buffer.array()); 
                // 输出该消息
                System.out.println("form 客户端: " + msg);
                // 向其它的客户端转发消息(去掉自己), 专门写一个方法来处理
                sendInfoToOtherClients(msg, channel);
            }

        } catch (IOException e) {
            try {
                System.out.println(channel.getRemoteAddress() + " 离线了.."); // 取消注册
                key.cancel(); // 关闭通道
                channel.close();
            } catch (IOException e2) {
                e2.printStackTrace();
            }
        }
    }

    // 转发消息给其它客户(通道)
    private void sendInfoToOtherClients(String msg, SocketChannel self) throws IOException {
        System.out.println("服务器转发消息中..."); 
        // 遍历 所有注册到 selector 上的 SocketChannel,并排除 self
        for (SelectionKey key : selector.keys()) {
            // 通过 key 取出对应的 SocketChannel
            Channel targetChannel = key.channel();
            // 排除自己
            if (targetChannel instanceof SocketChannel && targetChannel != self) {
                // 转型
                SocketChannel dest = (SocketChannel) targetChannel; // 将 msg 存储到 buffer
                ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes());
                // 将 buffer 的数据写入 通道
                dest.write(buffer);
            }
        }
    }

    public static void main(String[] args) {
        // 创建服务器对象
        GroupChatServer groupChatServer = new GroupChatServer();
        groupChatServer.listen();
    }
}
```

# GroupChatClient

```java
 package com.nio.chargroup;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-03-19 15:21:21
 * @LastEditTime: 2020-03-19 15:34:03
 * @LastEditors: 麦子
 */

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Scanner;

public class GroupChatClient {

    // 定义相关的属性
    private final String HOST = "127.0.0.1";
    // 服务器的 ip
    private final int PORT = 6667;
    // 服务器端口
    private final Selector selector;
    private final SocketChannel socketChannel;
    private final String username;

    public GroupChatClient() throws IOException {

        selector = Selector.open(); // 连接服务器
        socketChannel = SocketChannel.open(new InetSocketAddress(HOST, PORT));
        // 设置非阻塞
        socketChannel.configureBlocking(false);
        // 将 channel 注册到 selector
        socketChannel.register(selector, SelectionKey.OP_READ);
        // 得到 username
        username = socketChannel.getLocalAddress().toString().substring(1);
        System.out.println(username + " is ok...");

    }

    // 向服务器发送消息
    public void sendInfo(String info) {

        info = username + " 说：" + info;
        try {
            socketChannel.write(ByteBuffer.wrap(info.getBytes()));
        } catch (final IOException e) {
            e.printStackTrace();
        }
    }

    // 读取从服务器端回复的消息
    public void readInfo() {
        try {
            final int readChannels = selector.select();
            if (readChannels > 0) {
                // 有可以用的通道
                final Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                while (iterator.hasNext()) {
                    final SelectionKey key = iterator.next();
                    if (key.isReadable()) {
                        // 得到相关的通道
                        final SocketChannel sc = (SocketChannel) key.channel(); 
                        // 得到一个 Buffer
                        final ByteBuffer buffer = ByteBuffer.allocate(1024); 
                        // 读取
                        sc.read(buffer); 
                        // 把读到的缓冲区的数据转成字符串
                        final String msg = new String(buffer.array());
                        System.out.println(msg.trim());
                    }
                }
                // 删除当前的 selectionKey, 防止重复操作
                iterator.remove(); 
            } else {
                // System.out.println("没有可以用的通道...");
            }
        } catch (final Exception e) {
            e.printStackTrace();
        }

    }

    public static void main(final String[] args) throws Exception {
        final GroupChatClient chatClient = new GroupChatClient();
        // 启动一个线程, 每个 3 秒，读取从服务器发送数据
        new Thread() {
            public void run() {
                while (true) {
                    chatClient.readInfo();
                    try {
                        Thread.currentThread().sleep(3000);
                    } catch (final InterruptedException e) {
                        e.printStackTrace();
                    }

                }

            }
        }.start();

        // 发送数据给服务器端
        final Scanner scanner = new Scanner(System.in);
        while (scanner.hasNextLine()) {
            final String s = scanner.nextLine();
            chatClient.sendInfo(s);
        }

    }
}
```

# 运行结果

 ![Xnip2020-03-20_13-59-36](/img/Xnip2020-03-20_13-59-36.png)

# 说明

这里的群聊都是通过服务器转发消息的，但是这种转发是很消耗性能的，通过Netty有更好的处理方式，后期文章进行优化处理。 
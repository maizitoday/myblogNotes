---
title:       "NIO-SOCKET简单demo"
subtitle:    ""
description: ""
date:        2020-01-30
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["网络编程","java基础"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://juejin.im/post/5ae33c026fb9a07a9c03f45b**

# 服务端

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

public class ChannelServer {
    public static void main(String[] args) {
        Selector selector = null;
        try {
            // 1. 创建 Selector 实例
            selector = Selector.open();
            // 2. 创建 ServerSocketChannel 实例，配置为非阻塞模式，绑定本地端口。
            ServerSocketChannel ssc = ServerSocketChannel.open();
            ssc.configureBlocking(false);
            ssc.bind(new InetSocketAddress(8899));
            // 3. 把 ServerSocketChannel实例 注册到 Selector 实例中
            ssc.register(selector, SelectionKey.OP_ACCEPT);

            while (true) {
                // 4. 这里设置了3秒超时时间，也就是阻塞3秒
                if (selector.select(3000) == 0) {
                    continue;
                }
                // 5. 获取选中的 SelectionKey 的集合
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                Iterator<SelectionKey> iterator = selectionKeys.iterator();
                // 6. 处理 SelectionKey 的感兴趣的操作。注册到 selector 中的 serverSocketChannel 只能是
                // isAcceptable() ，因此通过它的 accept() 方法，
                // 我们可以获取到客户端的请求 SocketChannel 实例，然后再把这个 socketChannel 注册到 selector
                // 中，设置为可读的操作。那么下次遍历 selectionKeys 的时候，就可以处理那么可读的操作
                while (iterator.hasNext()) {
                    SelectionKey key = iterator.next();

                    // 准备好连接
                    if (key.isAcceptable()) {
                        ServerSocketChannel serverSocketChannel = (ServerSocketChannel) key.channel();
                        SocketChannel clientChannel = serverSocketChannel.accept();
                        clientChannel.configureBlocking(false);
                        clientChannel.register(key.selector(), SelectionKey.OP_READ);
                    }

                    // 准备好读的操作
                    if (key.isReadable()) {
                        SocketChannel clientChannel = (SocketChannel) key.channel();
                        ByteBuffer readBuffer = ByteBuffer.allocate(10);
                        int readBytes = clientChannel.read(readBuffer);
                        if (readBytes == -1) {
                            System.out.println("closed.......");
                            clientChannel.close();
                        } else if (readBytes > 0) {
                            String s = new String(readBuffer.array());
                            System.out.println("Client said: " + s);
                            if (s.trim().equalsIgnoreCase("Hello")) {
                                // attachment is content used to write
                                key.interestOps(SelectionKey.OP_WRITE);
                                key.attach("Welcome maizi !!!");
                            }
                        }
                    }

                    if (key.isValid() && key.isWritable()) {
                        SocketChannel clientChannel = (SocketChannel) key.channel();
                        String content = (String) key.attachment();
                        // write content to socket channel
                        clientChannel.write(ByteBuffer.wrap(content.getBytes()));
                        key.interestOps(SelectionKey.OP_READ);
                    }

                    // remove handled key from selected keys
                    iterator.remove();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // close selector
            if (selector != null) {
                try {
                    selector.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

```



# 客户端

```java
import java.io.IOException;
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.net.SocketException;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;

public class ChannelClient {
    public static void main(String[] args) {
        byte[] data = "hello".getBytes();
        SocketChannel channel = null;
        try {
            // 1. 创建 SocketChannel 实例，并配置为非阻塞模式，只有在非阻塞模式下，任何在 SocketChannel 实例上的 I/O
            // 操作才是非阻塞的。这样我们的客户端就是一个非阻塞式客户端，也就可以提升客户端性能。
            channel = SocketChannel.open();
            channel.configureBlocking(false);
            // 2. 用 connect() 方法连接服务器，同时用 while 循环不断检测并完全连接。 其实我们可以不用这样盲等，这里只是为了演示连接的过程。
            // 当你在需要马上进行 I/O 操作前，必须要用 finishConnect() 完成连接过程。
            if (!channel.connect(new InetSocketAddress(InetAddress.getLocalHost(), 8899))) {
                while (!channel.finishConnect()) {
                    System.out.print("正在尝试连接服务器.....");
                }
            }

            System.out.println("服务器连接成功!");

            ByteBuffer writeBuffer = ByteBuffer.wrap(data);
            ByteBuffer readBuffer = ByteBuffer.allocate(1024);
            int totalBytesReceived = 0;
            int bytesReceived;
            // 3. 用 ByteBuffer 读写字节，这里我们为何和一个 while 循环不断地读写呢？ 还记得前面讲 SelectableChannel
            // 非阻塞时的特性吗？ 如果一个 SelectableChannel 为非阻塞模式，它的 I/O 操作读写的字节数可能比实际的要少，甚至没有。
            // 所以我们这里用循环不断的读写，保证读写完成。
            while (totalBytesReceived < data.length) {
                if (writeBuffer.hasRemaining()) {
                    channel.write(writeBuffer);
                }
                if ((bytesReceived = channel.read(readBuffer)) == -1) {
                    throw new SocketException("Connection closed prematurely");
                }
                totalBytesReceived += bytesReceived;
                System.out.println("等待服务器回应.....");
            }
            System.out.println("Server said: " + new String(readBuffer.array()));
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // 4 .close socket channel
            try {
                if (channel != null) {
                    channel.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

```

# 运行结果

## 服务端

```java
Client said: hello
closed.......
Client said: hello
closed.......
Client said: hello
closed.......
Client said: hello
closed.......
Client said: hello
closed.......
Client said: hello
closed.......
```

## 客户端

```java
服务器连接成功!
等待服务器回应.....
Server said: Welcome maizi !!!
```


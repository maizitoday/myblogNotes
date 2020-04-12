---
title:       "Netty核心模块组件"
subtitle:    ""
description: "Netty-群聊系统, 心跳检测机制"
date:        2020-03-31
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["网络编程","Netty-群聊系统","心跳检测机制"]
categories:  ["Tech" ]
---

[TOC]

**说明文章资料来源：尚硅谷韩顺平Netty视频教程（2019发布） https://www.bilibili.com/video/av76227904?p=36**

# Bootstrap、ServerBootstrap

1) Bootstrap 意思是引导，一个 Netty 应用通常由一个 Bootstrap 开始，主要作用是配置整个 Netty 程序，串联 各个组件，Netty 中 Bootstrap 类是客户端程序的启动引导类，ServerBootstrap 是服务端启动引导类

2) 常见的方法有

```java
//该方法用于服务器端， 用来设置两个 EventLoop
public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup)，

//该方法用于客户端，用来设置一个 EventLoop
public B group(EventLoopGroup group) ，

//该方法用来设置一个服务器端的通道实现
public B channel(Class<? extends C> channelClass)，

//用来给 ServerChannel 添加配置 
public <T> B option(ChannelOption<T> option, T value)，

//用来给接收到的通道添加配置 
public <T> ServerBootstrap childOption(ChannelOption<T> childOption, T value)，

//该方法用来设置业务处理类（自定义的 handler） 
public ServerBootstrap childHandler(ChannelHandler childHandler)， 

//该方法用于服务器端，用来设置占用的端口号 
public ChannelFuture bind(int inetPort) ，

//该方法用于客户端，用来连接服务器端
public ChannelFuture connect(String inetHost, int inetPort) 
```

# Future、ChannelFuture

Netty 中所有的 IO 操作都是异步的，不能立刻得知消息是否被正确处理。但是可以过一会等它执行完成或 者直接注册一个监听，具体的实现就是通过 Future 和 ChannelFutures，他们可以注册一个监听，当操作执行成功 或失败时监听会自动触发注册的监听事件

常见的方法有

```java
#返回当前正在进行 IO 操作的通道
Channel channel()，

#等待异步操作执行完毕
ChannelFuture sync()，
```

# Channel

1) Netty 网络通信的组件，能够用于执行网络 I/O 操作。

2) 通过 Channel 可获得当前网络连接的通道的状态

3) 通过 Channel 可获得 网络连接的配置参数 （例如接收缓冲区大小）

4) Channel 提供异步的网络 I/O 操作(如建立连接，读写，绑定端口)，异步调用意味着任何 I/O 调用都将立即返 回，并且不保证在调用结束时所请求的 I/O 操作已完成

5) 调用立即返回一个 ChannelFuture 实例，通过注册监听器到 ChannelFuture 上，可以 I/O 操作成功、失败或取 消时回调通知调用方

6) 支持关联 I/O 操作与对应的处理程序

7) 不同协议、不同的阻塞类型的连接都有不同的 Channel 类型与之对应，常用的 Channel 类型:

- NioSocketChannel，异步的客户端 TCP Socket 连接。
- NioServerSocketChannel，异步的服务器端 TCP Socket 连接。
- NioDatagramChannel，异步的 UDP 连接。
- NioSctpChannel，异步的客户端 Sctp 连接。
- NioSctpServerChannel，异步的 Sctp 服务器端连接，这些通道涵盖了 UDP 和 TCP 网络 IO 以及文件 IO。



# Selector

1) Netty 基于 Selector 对象实现 I/O 多路复用，通过 Selector 一个线程可以监听多个连接的 Channel 事件。

2) 当向一个 Selector 中注册 Channel 后， Selector 内部的机制就可以自动不断地查询(Select) 这些注册的 Channel 是否有已就绪的 I/O 事件（例如可读，可写，网络连接完成等），这样程序就可以很简单地使用一个 线程高效地管理多个 Channel



# ChannelHandler 及其实现类

1) ChannelHandler 是一个接口，处理 I/O 事件或拦截 I/O 操作，并将其转发到其 ChannelPipeline(业务处理链) 中的下一个处理程序。

2) ChannelHandler 本身并没有提供很多方法，因为这个接口有许多的方法需要实现，方便使用期间，可以继承它 的子类

3) ChannelHandler 及其实现类一览图(后)

![Xnip2020-03-31_16-03-12](/img/Xnip2020-03-31_16-03-12.png)

4) 我们经常需要自定义一个 Handler 类去继承 ChannelInboundHandlerAdapter，然后通过重写相应方法实现业务 逻辑，我们接下来看看一般都需要重写哪些方法

![Xnip2020-03-31_16-04-26](/img/Xnip2020-03-31_16-04-26.png)



# Pipeline 和 ChannelPipeline

ChannelPipeline 是一个重点：

1) ChannelPipeline 是一个 Handler 的集合，它负责处理和拦截 inbound 或者 outbound 的事件和操作，相当于 一个贯穿 Netty 的链。(也可以这样理解：ChannelPipeline 是 保存 ChannelHandler 的 List，用于处理或拦截 Channel 的入站事件和出站操作)

2) ChannelPipeline 实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以及 Channel 中各个的 ChannelHandler 如何相互交互

3) 在 Netty 中每个 Channel 都有且仅有一个 ChannelPipeline 与之对应，它们的组成关系如下

![Xnip2020-03-31_16-06-58](/img/Xnip2020-03-31_16-06-58.png)

**从上面可以看到这就是一个链条，也可以说是业务经过这一系列的handler的处理。** 

4) 常用方法

```java
//把一个业务处理类（handler）添加到链中的第一个位置 
ChannelPipeline addFirst(ChannelHandler... handlers)，

//把一个业务处理类（handler）添加到链中的最后一个位置
ChannelPipeline addLast(ChannelHandler... handlers)，
```

代码展示

```java
// 使用链式编程来进行设置
    bootstrap.group(bossGroup, workerGroup) // 设置两个线程组
       .channel(NioServerSocketChannel.class) // 使用 NioSocketChannel 作为服务器的通道实现
       .option(ChannelOption.SO_BACKLOG, 128) // 设置线程队列得到连接个数
       .childOption(ChannelOption.SO_KEEPALIVE, true) // 设置保持活动连接状态
       .childHandler(new ChannelInitializer<SocketChannel>() {// 创建一个通道测试对象(匿名对象)
                        // 给 pipeline 设置处理器
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            
                            ch.pipeline().addLast(new NettyServerHandler());
                        }
                    }); // 给我们的 workerGroup 的 EventLoop 对应的管道设置处理器

```



# ChannelHandlerContext

```java
   // 数据读取完毕
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {

        // writeAndFlush 是 write + flush
        // 将数据写入到缓存，并刷新
        // 一般讲，我们对这个发送的数据进行编码
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵", CharsetUtil.UTF_8));

    }
```

1) 保存 Channel 相关的所有上下文信息，同时关联一个 ChannelHandler 对象

2) 即 ChannelHandlerContext 中 包 含 一 个 具 体 的 事 件 处 理 器 ChannelHandler ， 同 ChannelHandlerContext 中也绑定了对应的 pipeline 和 Channel 的信息，方便对 ChannelHandler 进行调用.

3) 常用方法

![Xnip2020-03-31_16-13-27](/img/Xnip2020-03-31_16-13-27.png)



# ChannelOption

```java
// 使用链式编程来进行设置
    bootstrap.group(bossGroup, workerGroup) // 设置两个线程组
       .channel(NioServerSocketChannel.class) // 使用 NioSocketChannel 作为服务器的通道实现
       .option(ChannelOption.SO_BACKLOG, 128) // 设置线程队列得到连接个数
       .childOption(ChannelOption.SO_KEEPALIVE, true) // 设置保持活动连接状态
       .childHandler(new ChannelInitializer<SocketChannel>() {// 创建一个通道测试对象(匿名对象)
                        // 给 pipeline 设置处理器
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            
                            ch.pipeline().addLast(new NettyServerHandler());
                        }
                    }); // 给我们的 workerGroup 的 EventLoop 对应的管道设置处理器
```

1) Netty 在创建 Channel 实例后,一般都需要设置 ChannelOption 参数。

2) ChannelOption 参数如下:

![Xnip2020-03-31_16-14-54](/img/Xnip2020-03-31_16-14-54.png)





# EventLoopGroup 和其实现类 NioEventLoopGroup

```java
// 创建 BossGroup 和 WorkerGroup
// 说明
// 1. 创建两个线程组 bossGroup 和 workerGroup
// 2. bossGroup 只是处理连接请求 , 真正的和客户端业务处理，会交给 workerGroup 完成
// 3. 两个都是无限循环
// 4. bossGroup 和 workerGroup 含有的子线程(NioEventLoop)的个数
// 默认实际 cpu 核数 * 2
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();
```

1) EventLoopGroup 是一组 EventLoop 的抽象，Netty 为了更好的利用多核 CPU 资源，一般会有多个 EventLoop 同时工作，每个 EventLoop 维护着一个 Selector 实例。

2) EventLoopGroup 提供 next 接口，可以从组里面按照一定规则获取其中一个 EventLoop 来处理任务。在 Netty 服 务 器 端 编 程 中 ， 我 们 一 般 都 需 要 提 供 两 个 EventLoopGroup ， 例 如 ： BossEventLoopGroup 和 WorkerEventLoopGroup。

3) 通常一个服务端口即一个 ServerSocketChannel 对应一个 Selector 和一个 EventLoop 线程。BossEventLoop 负责 接收客户端的连接并将 SocketChannel 交给 WorkerEventLoopGroup 来进行 IO 处理，如下图所示

![Xnip2020-03-31_16-17-11](/img/Xnip2020-03-31_16-17-11.png)

4) 常用方法

```java
// 构造方法 
public NioEventLoopGroup()
// 断开连接，关闭线程
public Future<?> shutdownGracefully()，
```



# Unpooled 类

```java
 // 数据读取完毕
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {

        // writeAndFlush 是 write + flush
        // 将数据写入到缓存，并刷新
        // 一般讲，我们对这个发送的数据进行编码
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵", CharsetUtil.UTF_8));

    }
```

1) Netty 提供一个专门用来操作缓冲区(即 Netty 的数据容器)的工具类

2) 常用方法如下所示

![Xnip2020-03-31_16-19-51](/img/Xnip2020-03-31_16-19-51.png)

3) 举例说明 Unpooled 获取 Netty 的数据容器 ByteBuf 的基本使用 【案例演示】

![Xnip2020-03-31_16-20-11](/img/Xnip2020-03-31_16-20-11.png)

**案例 1**

```java
package com.nettycode;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-03-31 16:22:36
 * @LastEditTime: 2020-03-31 16:28:26
 * @LastEditors: 麦子
 */

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;

public class NettyByteBuf01 {

    public static void main(String[] args) {
        // 创建一个 ByteBuf
        // 1. 创建 对象，该对象包含一个数组 arr , 是一个 byte[10]
        // 2. 在 netty 的 buffer 中，不需要使用 flip 进行反转 底层维护了 readerindex 和 writerIndex
        // 3. 通过 readerindex 和 writerIndex 和 capacity， 将 buffer 分成三个区域
        // 0---readerindex 已经读取的区域
        // readerindex---writerIndex ， 可读的区域
        // writerIndex -- capacity, 可写的区域

        ByteBuf buffer = Unpooled.buffer(10);
        for (int i = 0; i < 10; i++) {

            buffer.writeByte(i);
        }
        System.out.println("capacity=" + buffer.capacity());// 10

        for (int i = 0; i < buffer.capacity(); i++) {

            System.out.println(buffer.readByte());
        }
        System.out.println("执行完毕");
    }

}
```

**注意：它和NIO的ByteBuffer有区别，它是可以读和写的，不需要flip进行反转的。因为他的读和写下标是分开处理的**

# Netty 应用实例-群聊系统

## 实现需求

1) 编写一个 Netty 群聊系统，实现服务器端和客户端之间的数据简单通讯（非阻塞） 

2) 实现多人群聊 

3) 服务器端：可以监测用户上线，离线，并实现消息转发功能

4) 客户端：通过 channel 可以无阻塞发送消息给其它所有用户，同时可以接受其它用户发送的消息(有服务器转发 得到) 



## 代码

### GroupChatServer

```java
package com.nettycode.chartgroup;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-03-31 16:34:42
 * @LastEditTime: 2020-03-31 16:43:21
 * @LastEditors: 麦子
 */

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;

public class GroupChatServer {
    private int port; // 监听端口

    public GroupChatServer(int port) {

        this.port = port;
    }

    // 编写 run 方法，处理客户端的请求
    public void run() throws Exception {

        // 创建两个线程组
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup(); // 8 个 NioEventLoop

        try {

            ServerBootstrap b = new ServerBootstrap();

            b.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class).option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    .childHandler(new ChannelInitializer<SocketChannel>() {

                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {

                            // 获取到 pipeline
                            ChannelPipeline pipeline = ch.pipeline();
                            // 向 pipeline 加入解码器
                            pipeline.addLast("decoder", new StringDecoder());
                            // 向 pipeline 加入编码器
                            pipeline.addLast("encoder", new StringEncoder());
                            // 加入自己的业务处理 handler
                            pipeline.addLast(new GroupChatServerHandler());

                        }

                    });

            System.out.println("netty 服务器启动");
            ChannelFuture channelFuture = b.bind(port).sync();

            // 监听关闭
            channelFuture.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        new GroupChatServer(7000).run();
    }
}
```

### GroupChatServerHandler

```java
package com.nettycode.chartgroup;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-03-31 16:43:29
 * @LastEditTime: 2020-03-31 17:02:29
 * @LastEditors: 麦子
 */

import java.text.SimpleDateFormat;

import io.netty.channel.Channel;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.group.ChannelGroup;
import io.netty.channel.group.DefaultChannelGroup;
import io.netty.util.concurrent.GlobalEventExecutor;

public class GroupChatServerHandler extends SimpleChannelInboundHandler<String> {

    // GlobalEventExecutor.INSTANCE) 是全局的事件执行器，是一个单例
    private static ChannelGroup channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    // handlerAdded 表示连接建立，一旦连接，第一个被执行
    // 将当前 channel 加入到 channelGroup
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        // 将该客户加入聊天的信息推送给其它在线的客户端
        /* 该方法会将 channelGroup 中所有的 channel 遍历，并发送 消息， 我们不需要自己遍历 */
        channelGroup.writeAndFlush(
                "[ 客 户 端 ]" + channel.remoteAddress() + " 加 入 聊 天 " + sdf.format(new java.util.Date()) + " \n");
        channelGroup.add(channel);
    }

    // 断开连接, 将 xx 客户离开信息推送给当前在线的客户
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        channelGroup.writeAndFlush("[客户端]" + channel.remoteAddress() + " 离开了\n");
        System.out.println("channelGroup size" + channelGroup.size());

    }

    // 表示 channel 处于活动状态, 提示 xx 上线
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println(ctx.channel().remoteAddress() + " 上线了~");
    }

    // 表示 channel 处于不活动状态, 提示 xx 离线了
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.out.println(ctx.channel().remoteAddress() + " 离线了~");
    }

    // 读取数据
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        // 获取到当前 channel
        Channel channel = ctx.channel();
        // 这时我们遍历 channelGroup, 根据不同的情况，回送不同的消息
        channelGroup.forEach(ch -> {
            if (channel != ch) { // 不是当前的 channel,转发消息

                ch.writeAndFlush("[客户]" + channel.remoteAddress() + " 发送了消息:  " + msg + "\n");
            } else {// 回显自己发送的消息给自己

                ch.writeAndFlush("[自己]发送了消息:  " + msg + "\n");
            }

        });

    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        // 关闭通道
        ctx.close();
    }
}
```

### GroupChatClient

```java
package com.nettycode.chartgroup;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-03-31 16:50:40
 * @LastEditTime: 2020-03-31 16:56:30
 * @LastEditors: 麦子
 */

import java.util.Scanner;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;

public class GroupChatClient {
    // 属性
    private final String host;
    private final int port;

    public GroupChatClient(final String host, final int port) {
        this.host = host;
        this.port = port;
    }

    public void run() throws Exception {
        final EventLoopGroup group = new NioEventLoopGroup();
        try {
            final Bootstrap bootstrap = new Bootstrap().group(group).channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {

                        @Override
                        protected void initChannel(final SocketChannel ch) throws Exception {

                            // 得到 pipeline
                            final ChannelPipeline pipeline = ch.pipeline();
                            // 加入相关 handler
                            pipeline.addLast("decoder", new StringDecoder());
                            pipeline.addLast("encoder", new StringEncoder());
                            // 加入自定义的 handler
                            pipeline.addLast(new GroupChatClientHandler());

                        }

                    });

            final ChannelFuture channelFuture = bootstrap.connect(host, port).sync();
            // 得到 channel
            final Channel channel = channelFuture.channel();
            System.out.println("-------" + channel.localAddress() + "--------");
            // 客户端需要输入信息，创建一个扫描器
            Scanner scanner = new Scanner(System.in);
            while (scanner.hasNextLine()) {
                final String msg = scanner.nextLine();
                // 通过 channel 发送到服务器端
                channel.writeAndFlush(msg + "\r\n");
            }

        } catch (final Exception e) {
            e.printStackTrace();
        } finally {
            group.shutdownGracefully();
        }
    }

    public static void main(final String[] args) throws Exception {

        new GroupChatClient("127.0.0.1", 7000).run();
    }
}
```

### GroupChatClientHandler

```java
package com.nettycode.chartgroup;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-03-31 16:55:07
 * @LastEditTime: 2020-03-31 16:55:59
 * @LastEditors: 麦子
 */

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class GroupChatClientHandler extends SimpleChannelInboundHandler<String> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        
        System.out.println(msg.trim());

    }

}
```

## 运行效果

### 上线聊天效果

![Xnip2020-03-31_17-05-05](/img/Xnip2020-03-31_17-05-05.png)

### 离线，离开效果

![Xnip2020-03-31_17-05-21](/img/Xnip2020-03-31_17-05-21.png)

## 注意

**可以看到 channelGroup 的size大小**

# Netty 心跳检测机制案例

## 实现需求

1) 编写一个 Netty 心跳检测机制案例, 当服务器超过 3 秒没有读时，就提示读空闲

2) 当服务器超过 5 秒没有写操作时，就提示写空闲

3) 实现当服务器超过 7 秒没有读或者写操作时，就提示读写空闲

## 代码

### MyServer

```java
package com.nettycode.heartbeat;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-03-31 17:13:26
 * @LastEditTime: 2020-03-31 17:34:27
 * @LastEditors: 麦子
 */

import java.util.concurrent.TimeUnit;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import io.netty.handler.timeout.IdleStateHandler;

public class MyServer {

    public static void main(String[] args) {
        // 创建两个线程组
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup);
            serverBootstrap.channel(NioServerSocketChannel.class);
            // 加入日志
            serverBootstrap.handler(new LoggingHandler(LogLevel.INFO));
            serverBootstrap.childHandler(new ChannelInitializer<SocketChannel>() {

                @Override
                protected void initChannel(SocketChannel ch) throws Exception {
                    ChannelPipeline pipeline = ch.pipeline();
                    // 加入一个 netty 提供 IdleStateHandler
                    /*
                     * 说明 
                     * 1. IdleStateHandler 是 netty 提供的处理空闲状态的处理器 
                     * 2. long readerIdleTime : 表示多长时间没有读, 就会发送一个心跳检测包检测是否连接
                     * 3. long writerIdleTime : 表示多长时间没有写, 就会发送一个心跳检测包检测是否连接 
                     * 4. long allIdleTime : 表示多长时间没有读写, 就会发送一个心跳检测包检测是否连接
                     * 5. 当 IdleStateEvent 触发后 , 就会传递给管道 的下一个 handler 去处理 * 通过调用(触发)下一个 handler 的 userEventTiggered , 在该方法中去处理 IdleStateEvent(读 空闲，写空闲，读写空闲)
                     */
                    pipeline.addLast(new IdleStateHandler(13,5,2, TimeUnit.SECONDS));
                    //加入一个对空闲检测进一步处理的 handler(自定义)
                    pipeline.addLast(new MyServerHandler());
                }

            });

            //启动服务器 
            ChannelFuture channelFuture = serverBootstrap.bind(7000).sync(); 
            channelFuture.channel().closeFuture().sync();

        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            bossGroup.shutdownGracefully(); 
            workerGroup.shutdownGracefully();  
        }
    }

}
```

### MyServerHandler

```java
package com.nettycode.heartbeat;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-03-31 17:20:08
 * @LastEditTime: 2020-03-31 17:25:33
 * @LastEditors: 麦子
 */

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.handler.timeout.IdleStateEvent;

public class MyServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if (evt instanceof IdleStateEvent) {
            // 将 evt 向下转型 IdleStateEvent
            IdleStateEvent event = (IdleStateEvent) evt;
            String eventType = null;
            switch (event.state()) {

                case READER_IDLE:
                    eventType = "读空闲";
                    break;
                case WRITER_IDLE:
                    eventType = "写空闲";
                    break;
                case ALL_IDLE:
                    eventType = "读写空闲";
                    break;
            }
            System.out.println(ctx.channel().remoteAddress() + "--超时时间--" + eventType);
            System.out.println("服务器做相应处理..");
            // 如果发生空闲，我们关闭通道
            // ctx.channel().close();
        }

    }
}
```

然后在用上面的 GroupChatClient 的类启动客户端。

## 运行效果

```java
/127.0.0.1:54542--超时时间--读写空闲
服务器做相应处理..
/127.0.0.1:54542--超时时间--读写空闲
服务器做相应处理..
/127.0.0.1:54542--超时时间--写空闲
服务器做相应处理..
/127.0.0.1:54542--超时时间--读写空闲
服务器做相应处理..
/127.0.0.1:54542--超时时间--读写空闲
服务器做相应处理..
/127.0.0.1:54542--超时时间--写空闲
服务器做相应处理..
/127.0.0.1:54542--超时时间--读写空闲
```




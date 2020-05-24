---
title:       "Netty-RPC"
subtitle:    ""
description: "RPC概念，常用RPC框架，Netty自定义RPC框架"
date:        2020-05-24
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["网络编程"]
categories:  ["Tech" ]
---

[TOC]

# RPC基本介绍

RPC（Remote Procedure Call）— 远程过程调用，是一个计算机通信协议。该协议允许运行于一台计算机的程序调用另一台计算机的子程序，而程序员无需额外地为这个交互作用编程

两个或多个应用程序都分布在不同的服务器上，它们之间的调用都像是本地方法调用一样(如图)

![Xnip2020-05-24_09-42-21](/img/Xnip2020-05-24_09-42-21.png)

# 常见的 RPC 框架有

比较知名的如阿里的Dubbo、google的gRPC、Go语言的rpcx、Apache的thrift， Spring 旗下的 Spring Cloud。



# RPC调用流程

![Xnip2020-05-24_09-43-46](/img/Xnip2020-05-24_09-43-46.png)



## 调用流程说明

1. **服务消费方(client)以本地调用方式调用服务**
2. client stub 接收到调用后负责将方法、参数等封装成能够进行网络传输的消息体
3. client stub 将消息进行编码并发送到服务端
4. server stub 收到消息后进行解码
5. server stub 根据解码结果调用本地的服务
6. 本地服务执行并将结果返回给 server stub
7. server stub 将返回导入结果进行编码并发送至消费方
8. client stub 接收到消息并进行解码
9. **服务消费方(client)得到结果**

**小结：RPC 的目标就是将 2-8 这些步骤都封装起来，用户无需关心这些细节，可以像调用本地方法一样即可完成远程服务调用。**



# 使用Netty编写Doubbo RPC

## 思路流程

![Xnip2020-05-21_09-48-57](/img/Xnip2020-05-21_09-48-57.png)



## 具体代码如下



### 公共接口，也就是要消费和提供的接口 

```java
package com.maizi.rpc.publicinterface;

//这个是接口，是服务提供方和 服务消费方都需要
public interface HelloService {

    String hello(String mes);
}

```



### 基础类封装

#### NettyClient

```java
package com.maizi.rpc.netty;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;

import java.lang.reflect.Proxy;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class NettyClient {

    // 创建线程池
    private static ExecutorService executor = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());

    private static NettyClientHandler client;
    private int count = 0;

    // 编写方法使用代理模式，获取一个代理对象

    public Object getBean(final Class<?> serivceClass, final String providerName) {

        return Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), new Class<?>[] { serivceClass },
                (proxy, method, args) -> {

                    System.out.println("(proxy, method, args) 进入...." + (++count) + " 次");
                    // {} 部分的代码，客户端每调用一次 hello, 就会进入到该代码
                    if (client == null) {
                        initClient();
                    }

                    // 设置要发给服务器端的信息
                    // providerName 协议头 args[0] 就是客户端调用api hello(???), 参数
                    client.setPara(providerName + args[0]);

                    //
                    return executor.submit(client).get();

                });
    }

    // 初始化客户端
    private static void initClient() {
        client = new NettyClientHandler();
        // 创建EventLoopGroup
        NioEventLoopGroup group = new NioEventLoopGroup();
        Bootstrap bootstrap = new Bootstrap();
        bootstrap.group(group).channel(NioSocketChannel.class).option(ChannelOption.TCP_NODELAY, true)
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        ChannelPipeline pipeline = ch.pipeline();
                        pipeline.addLast(new StringDecoder());
                        pipeline.addLast(new StringEncoder());
                        pipeline.addLast(client);
                    }
                });

        try {
            bootstrap.connect("127.0.0.1", 7000).sync();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

#### NettyClientHandler

```java
package com.maizi.rpc.netty;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

import java.util.concurrent.Callable;

public class NettyClientHandler extends ChannelInboundHandlerAdapter implements Callable {

    private ChannelHandlerContext context;// 上下文
    private String result; // 返回的结果
    private String para; // 客户端调用方法时，传入的参数

    // 与服务器的连接创建后，就会被调用, 这个方法是第一个被调用(1)
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println(" channelActive 被调用  ");
        context = ctx; // 因为我们在其它方法会使用到 ctx
    }

    // 收到服务器的数据后，调用方法 (4)
    //
    @Override
    public synchronized void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println(" channelRead 被调用  ");
        result = msg.toString();
        notify(); // 唤醒等待的线程
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }

    // 被代理对象调用, 发送数据给服务器，-> wait -> 等待被唤醒(channelRead) -> 返回结果 (3)-》5
    @Override
    public synchronized Object call() throws Exception {
        System.out.println(" call1 被调用  ");
        context.writeAndFlush(para);
        // 进行wait
        wait(); // 等待channelRead 方法获取到服务器的结果后，唤醒
        System.out.println(" call2 被调用  ");
        return result; // 服务方返回的结果

    }

    // (2)
    void setPara(String para) {
        System.out.println(" setPara  ");
        this.para = para;
    }
}

```



#### NettyServer

```java
package com.maizi.rpc.netty;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;

public class NettyServer {

    public static void startServer(String hostName, int port) {
        startServer0(hostName, port);
    }

    // 编写一个方法，完成对NettyServer的初始化和启动

    private static void startServer0(String hostname, int port) {

        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {

            ServerBootstrap serverBootstrap = new ServerBootstrap();

            serverBootstrap.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();
                            pipeline.addLast(new StringDecoder());
                            pipeline.addLast(new StringEncoder());
                            pipeline.addLast(new NettyServerHandler()); // 业务处理器

                        }
                    }

                    );

            ChannelFuture channelFuture = serverBootstrap.bind(hostname, port).sync();
            System.out.println("服务提供方开始提供服务~~");
            channelFuture.channel().closeFuture().sync();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }

    }
}

```

#### NettyServerHandler

```java
package com.maizi.rpc.netty;

import com.maizi.rpc.customer.ClientBootstrap;
import com.maizi.rpc.provider.HelloServiceImpl;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

//服务器这边handler比较简单
public class NettyServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 获取客户端发送的消息，并调用服务
        System.out.println("msg=" + msg);
        // 客户端在调用服务器的api 时，我们需要定义一个协议
        // 比如我们要求 每次发消息是都必须以某个字符串开头 "HelloService#hello#你好"
        if (msg.toString().startsWith(ClientBootstrap.providerName)) {

            String result = new HelloServiceImpl().hello(msg.toString().substring(msg.toString().lastIndexOf("#") + 1));
            ctx.writeAndFlush(result);
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}

```



### 消费接口类

```java
package com.maizi.rpc.customer;

import com.maizi.rpc.netty.NettyClient;
import com.maizi.rpc.publicinterface.HelloService;

public class ClientBootstrap {

    // 这里定义协议头
    public static final String providerName = "HelloService#hello#";

    public static void main(String[] args) throws Exception {

        // 创建一个消费者
        NettyClient customer = new NettyClient();

        // 创建代理对象
        HelloService service = (HelloService) customer.getBean(HelloService.class, providerName);

        for (;;) {
            Thread.sleep(2 * 1000);
            // 通过代理对象调用服务提供者的方法(服务)
            String res = service.hello("你好 dubbo~");
            System.out.println("调用的结果 res= " + res);
        }
    }
}

```



### 提供接口类

```java
package com.maizi.rpc.provider;

import com.maizi.rpc.publicinterface.HelloService;

public class HelloServiceImpl implements HelloService {

    private static int count;

    // 当有消费方调用该方法时， 就返回一个结果
    @Override
    public synchronized String hello(String mes) {
        System.out.println("收到客户端消息=" + mes);
        // 根据mes 返回不同的结果
        if (mes != null) {
            return "你好客户端, 我已经收到你的消息 [" + mes + "] 第" + (++count) + " 次";
        } else {
            return "你好客户端, 我已经收到你的消息 ";
        }
    }
}

```

```java
package com.maizi.rpc.provider;

import com.maizi.rpc.netty.NettyServer;

//ServerBootstrap 会启动一个服务提供者，就是 NettyServer
public class ServerBootstrap {
    public static void main(String[] args) {

        // 代码代填..
        NettyServer.startServer("127.0.0.1", 7000);
    }
}

```



### 运行结果

```java
服务提供方开始提供服务~~
msg=HelloService#hello#你好 dubbo~
收到客户端消息=你好 dubbo~
msg=HelloService#hello#你好 dubbo~
收到客户端消息=你好 dubbo~
msg=HelloService#hello#你好 dubbo~
收到客户端消息=你好 dubbo~
msg=HelloService#hello#你好 dubbo~
收到客户端消息=你好 dubbo~
msg=HelloService#hello#你好 dubbo~
```

```java
(proxy, method, args) 进入....1 次
 channelActive 被调用  
 setPara  
 call1 被调用  
 channelRead 被调用  
 call2 被调用  
调用的结果 res= 你好客户端, 我已经收到你的消息 [你好 dubbo~] 第1 次
(proxy, method, args) 进入....2 次
 setPara  
 call1 被调用  
 channelRead 被调用  
 call2 被调用  
调用的结果 res= 你好客户端, 我已经收到你的消息 [你好 dubbo~] 第2 次
(proxy, method, args) 进入....3 次
 setPara  
 call1 被调用  
 channelRead 被调用  
 call2 被调用  
调用的结果 res= 你好客户端, 我已经收到你的消息 [你好 dubbo~] 第3 次
```

# 总结

通过代理，执行公共接口的方法。

```java

    public Object getBean(final Class<?> serivceClass, final String providerName) {

        return Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), new Class<?>[] { serivceClass },
                (proxy, method, args) -> {

                    System.out.println("(proxy, method, args) 进入...." + (++count) + " 次");
                    // {} 部分的代码，客户端每调用一次 hello, 就会进入到该代码
                    if (client == null) {
                        initClient();
                    }

                    // 设置要发给服务器端的信息
                    // providerName 协议头 args[0] 就是客户端调用api hello(???), 参数
                    client.setPara(providerName + args[0]);

                    //
                    return executor.submit(client).get();

                });
    }
```

同时，实现Callable接口，

```java
public class NettyClientHandler extends ChannelInboundHandlerAdapter implements Callable {
```

回调下面的call方法。

```java
 // 被代理对象调用, 发送数据给服务器，-> wait -> 等待被唤醒(channelRead) -> 返回结果 (3)-》5
    @Override
    public synchronized Object call() throws Exception {
        System.out.println(" call1 被调用  ");
        context.writeAndFlush(para);
        // 进行wait
        wait(); // 等待channelRead 方法获取到服务器的结果后，唤醒
        System.out.println(" call2 被调用  ");
        return result; // 服务方返回的结果

    }

```


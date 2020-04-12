---
title:       "Netty编码和解码--Protobuf"
subtitle:    ""
description: "Protobuf传输，Protobuf取代JSON格式"
date:        2020-03-31
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["网络编程"]
categories:  ["Tech" ]
---

[TOC]

**说明文章资料来源：尚硅谷韩顺平Netty视频教程（2019发布） https://www.bilibili.com/video/av76227904?p=36**

# 编码和解码的基本介绍

1) 编写网络应用程序时，因为数据在网络中传输的都是二进制字节码数据，在发送数据时就需要编码，接收数据 时就需要解码 [示意图]

2) codec(编解码器) 的组成部分有两个：decoder(解码器)和 encoder(编码器)。encoder 负责把业务数据转换成字节 码数据，decoder 负责把字节码数据转换成业务数据

![Xnip2020-04-01_17-18-08](/img/Xnip2020-04-01_17-18-08.png)

# Netty 本身的编码解码的机制和问题分析

1) Netty 自身提供了一些 codec(编解码器)

2) Netty 提供的编码器  

​      StringEncoder：对字符串数据进行编码

​      ObjectEncoder：对 Java 对象进行编码

3) Netty 提供的解码器

​     StringDecoder, 对字符串数据进行解码

​     ObjectDecoder，对 Java 对象进行解码

**自带缺点**

Netty 本身自带的 ObjectDecoder 和 ObjectEncoder 可以用来实现 POJO 对象或各种业务对象的编码和解码，底层使用的仍是 Java 序列化技术 , 而 Java 序列化技术本身效率就不高，存在如下问题

- 无法跨语言
- 序列化后的体积太大，是二进制编码的 5 倍多
- 序列化性能太低

# Protobuf

原文链接：https://blog.csdn.net/love666666shen/article/details/89228450

原文地址：https://www.jianshu.com/p/bb3ac7e5834e

## 简介

protobuf是Google开发出来的一个语言无关、平台无关的数据序列化工具，在rpc或tcp通信等很多场景都可以使用。通俗来讲，如果客户端和服务端使用的是不同的语言，那么在服务端定义一个数据结构，通过protobuf转化为字节流，再传送到客户端解码，就可以得到对应的数据结构。这就是protobuf神奇的地方。并且，它的通信效率极高，“一条消息数据，用protobuf序列化后的大小是json的10分之一，xml格式的20分之一，是二进制序列化的10分之一”

支持跨平台、跨语言，即[客户端和服务器端可以是不同的语言编写的] （支持目前绝大多数语言，例如 C++、 C#、Java、python 等）

在一些场景下，数据需要在不同的平台，不同的程序中进行传输和使用，例如某个消息是用C++程序产生的，而另一个程序是用java写的，当前者产生一个消息数据时，需要在不同的语言编写的不同的程序中进行操作，如何将消息发送并在各个程序中使用呢？这就需要设计一种消息格式，常用的就有json和xml，protobuf出现的则较晚。



## 优点

- protobuf 的主要有点是简单，快。
- protobuf将数据序列化为二进制之后，占用的空间相当小，基本仅保留了数据部分，而xml和json会附带消息结构在数据中。
- protobuf使用起来很方便，只需要反序列化就可以了，而不需要xml和json那样层层解析。



## 与json的比较

虽然Json用起来的确很方便，但相对于protobuf数据量更大些。无论是移动端还是PC端应用，为用户省点流量还是很有必要的，减少数据传输量不仅可以节约带宽而且可以更快地得到响应，提升用户体验。



## 相比json的优点

- 跟Json相比protobuf性能更高，更加规范
- 编解码速度快，数据体积小
- 使用统一的规范，不用再担心大小写不同导致解析失败等问题



## 相比json的劣势

- 改动协议字段，需要重新生成文件。
- 数据没有可读性



## mac安装protobuf

使用brew命令进行protobuf安装默认安装最新的版本

```
brew install protobuf

#查看protoc版本
lcc@lcc ~$ protoc --version
libprotoc 3.6.0
```

## pom.xml 安装

```xml
<dependency>
      <groupId>com.google.protobuf</groupId>
      <artifactId>protobuf-java</artifactId>
      <version>3.7.0</version>
</dependency>
```

**注意：这个版本要和你安装的protobuf的版本要是一致的。** 



## .proto文件和Java字段转换

![Xnip2020-04-03_09-42-07](/img/Xnip2020-04-03_09-42-07.png)

## 编写.proto文件

```java
syntax = "proto3";

option java_package = "com.nettycode"; 
option java_outer_classname = "GpsDataProto";

message gps_data {
   int64 id = 1;
    string terminalId = 2;
    string dataTime = 3;
    double lon = 4;
    double lat = 5;
    float speed = 6;
    int32 altitude = 7;
    int32 locType = 8;
    int32 gpsStatus = 9;
    float direction = 10;
    int32 satellite = 11;
}
```

## 编译生成 Java 代码

- -I 后面是 proto 文件所在目录
- --java_out 后面是 java 文件存放地址
- 最后一行是 proto 文件名称

```shell
protoc -I=/vscode/高并发和网络编程/nettydemo/src/main/java/com/nettycode/proto --java_out=/vscode/高并发和网络编程/nettydemo/src/main/java/com/nettycode gps_data.proto
```

**注意：这里的地址要是绝对地址。** 



## 使用生成的Java代码

```java
package com.nettycode;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-04-03 10:04:27
 * @LastEditTime: 2020-04-03 10:05:28
 * @LastEditors: 麦子
 */

import com.google.protobuf.InvalidProtocolBufferException;

public class GPSDataProtoTest {

    public static void main(String[] args) {
        System.out.println("===== 构建一个GPS模型开始 =====");
        GpsDataProto.gps_data.Builder gps_builder = GpsDataProto.gps_data.newBuilder();
        gps_builder.setAltitude(1);
        gps_builder.setDataTime("2017-12-17 16:21:44");
        gps_builder.setGpsStatus(1);
        gps_builder.setLat(39.123);
        gps_builder.setLon(120.112);
        gps_builder.setDirection(30.2F);
        gps_builder.setId(100L);

        GpsDataProto.gps_data gps_data = gps_builder.build();
        System.out.println(gps_data.toString());
        System.out.println("===== 构建GPS模型结束 =====");

        System.out.println("===== gps Byte 开始=====");
        for (byte b : gps_data.toByteArray()) {
            System.out.print(b);
        }
        System.out.println("\n" + "bytes长度" + gps_data.toByteString().size());
        System.out.println("===== gps Byte 结束 =====");

        System.out.println("===== 使用gps 反序列化生成对象开始 =====");
        GpsDataProto.gps_data gd = null;
        try {
            gd = GpsDataProto.gps_data.parseFrom(gps_data.toByteArray());
        } catch (InvalidProtocolBufferException e) {
            e.printStackTrace();
        }
        System.out.print(gd.toString());
        System.out.println("===== 使用gps 反序列化生成对象结束 =====");
    }
    
}
```

## 运行结果

```java
===== 构建一个GPS模型开始 =====
id: 100
dataTime: "2017-12-17 16:21:44"
lon: 120.112
lat: 39.123
altitude: 1
gpsStatus: 1
direction: 30.2

===== 构建GPS模型结束 =====
===== gps Byte 开始=====
810026195048495545495045495532495458504958525233-707312243794644157-76-56118-66-113676456172185-102-103-1565
bytes长度50
===== gps Byte 结束 =====
===== 使用gps 反序列化生成对象开始 =====
id: 100
dataTime: "2017-12-17 16:21:44"
lon: 120.112
lat: 39.123
altitude: 1
gpsStatus: 1
direction: 30.2
===== 使用gps 反序列化生成对象结束 =====
```

# Protobuf 快速入门实例



## Student.proto

```java
syntax = "proto3"; //版本
option java_outer_classname = "StudentPOJO";//生成的外部类名，同时也是文件名
//protobuf 使用message 管理数据
message Student { //会在 StudentPOJO 外部类生成一个内部类 Student， 他是真正发送的POJO对象
    int32 id = 1; // Student 类中有 一个属性 名字为 id 类型为int32(protobuf类型) 1表示属性序号，不是值
    string name = 2;
}
```

## NettyServer

```java
package com.atguigu.netty.codec;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2019-10-28 11:40:46
 * @LastEditTime: 2020-04-03 10:15:36
 * @LastEditors: 麦子
 */

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.protobuf.ProtobufDecoder;

public class NettyServer {
    public static void main(String[] args) throws Exception {

        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup(); // 8

        try {

            ServerBootstrap bootstrap = new ServerBootstrap();

            bootstrap.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 128) // 设置线程队列得到连接个数
                    .childOption(ChannelOption.SO_KEEPALIVE, true) // 设置保持活动连接状态
                    // .handler(null) // 该 handler对应 bossGroup , childHandler 对应 workerGroup
                    .childHandler(new ChannelInitializer<SocketChannel>() {// 创建一个通道初始化对象(匿名对象)
                        // 给pipeline 设置处理器
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();
                            // 在pipeline加入ProtoBufDecoder
                            // 指定对哪种对象进行解码
                            pipeline.addLast("decoder", new ProtobufDecoder(StudentPOJO.Student.getDefaultInstance()));
                            pipeline.addLast(new NettyServerHandler());
                        }
                    }); // 给我们的workerGroup 的 EventLoop 对应的管道设置处理器

            System.out.println(".....服务器 is ready...");

            // 绑定一个端口并且同步, 生成了一个 ChannelFuture 对象
            // 启动服务器(并绑定端口)
            ChannelFuture cf = bootstrap.bind(6668).sync();

            // 给cf 注册监听器，监控我们关心的事件

            cf.addListener(new ChannelFutureListener() {
                @Override
                public void operationComplete(ChannelFuture future) throws Exception {
                    if (cf.isSuccess()) {
                        System.out.println("监听端口 6668 成功");
                    } else {
                        System.out.println("监听端口 6668 失败");
                    }
                }
            });

            // 对关闭通道进行监听
            cf.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }

    }

}

```

## NettyServerHandler

```java
package com.atguigu.netty.codec;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2019-10-28 11:44:22
 * @LastEditTime: 2020-04-03 10:16:57
 * @LastEditors: 麦子
 */

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.*;
import io.netty.util.CharsetUtil;

public class NettyServerHandler extends SimpleChannelInboundHandler<StudentPOJO.Student> {

    // 读取数据实际(这里我们可以读取客户端发送的消息)
    /*
     * 1. ChannelHandlerContext ctx:上下文对象, 含有 管道pipeline , 通道channel, 地址 2. Object
     * msg: 就是客户端发送的数据 默认Object
     */
    @Override
    public void channelRead0(ChannelHandlerContext ctx, StudentPOJO.Student msg) throws Exception {

        // 读取从客户端发送的StudentPojo.Student
        System.out.println("客户端发送的数据 id=" + msg.getId() + " 名字=" + msg.getName());
    }


    // 数据读取完毕
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {

        // writeAndFlush 是 write + flush
        // 将数据写入到缓存，并刷新
        // 一般讲，我们对这个发送的数据进行编码
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵1", CharsetUtil.UTF_8));
    }

    // 处理异常, 一般是需要关闭通道

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}

```

## NettyClient

```java
package com.atguigu.netty.codec;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2019-10-28 11:36:14
 * @LastEditTime: 2020-04-03 10:17:42
 * @LastEditors: 麦子
 */

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.protobuf.ProtobufEncoder;

public class NettyClient {
    public static void main(String[] args) throws Exception {

        EventLoopGroup group = new NioEventLoopGroup();

        try {

            Bootstrap bootstrap = new Bootstrap();

            // 设置相关参数
            bootstrap.group(group) // 设置线程组
                    .channel(NioSocketChannel.class) // 设置客户端通道的实现类(反射)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();
                            // 在pipeline中加入 ProtoBufEncoder
                            pipeline.addLast("encoder", new ProtobufEncoder());
                            pipeline.addLast(new NettyClientHandler()); // 加入自己的处理器
                        }
                    });

            System.out.println("客户端 ok..");

            // 启动客户端去连接服务器端
            // 关于 ChannelFuture 要分析，涉及到netty的异步模型
            ChannelFuture channelFuture = bootstrap.connect("127.0.0.1", 6668).sync();
            // 给关闭通道进行监听
            channelFuture.channel().closeFuture().sync();
        } finally {

            group.shutdownGracefully();

        }
    }
}

```

## NettyClientHandler

```java
package com.atguigu.netty.codec;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.util.CharsetUtil;

public class NettyClientHandler extends ChannelInboundHandlerAdapter {

    // 当通道就绪就会触发该方法
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {

        // 发生一个Student 对象到服务器

        StudentPOJO.Student student = StudentPOJO.Student.newBuilder().setId(4).setName("智多星 吴用").build();
        // Teacher , Member ,Message
        ctx.writeAndFlush(student);
    }

    // 当通道有读取事件时，会触发
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

        ByteBuf buf = (ByteBuf) msg;
        System.out.println("服务器回复的消息:" + buf.toString(CharsetUtil.UTF_8));
        System.out.println("服务器的地址： " + ctx.channel().remoteAddress());
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}

```

## 总结

可以看到在编码和解码的时候都指定了这个protobuf生成的java文件类型，这里的ProtobufEncoder，ProtobufDecoder

```java
 pipeline.addLast("encoder", new ProtobufEncoder());
 pipeline.addLast("decoder", new ProtobufDecoder(StudentPOJO.Student.getDefaultInstance()));
```

都是Netty包里面自带的类。 






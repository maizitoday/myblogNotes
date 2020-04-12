---
title:       "Netty-TCP粘包和拆包及解决方案"
subtitle:    ""
description: "粘包问题重现，解决粘包问题"
date:        2020-04-03
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["网络编程"]
categories:  ["Tech" ]
---

[TOC]

**说明文章资料来源：尚硅谷韩顺平Netty视频教程（2019发布） https://www.bilibili.com/video/av76227904?p=36**

**可以查看文章《TCP粘包和消息不完整》**

# TCP 粘包和拆包基本介绍

1) TCP 是面向连接的，面向流的，提供高可靠性服务。收发两端（客户端和服务器端）都要有一一成对的 socket， 因此，发送端为了将多个发给接收端的包，更有效的发给对方，使用了优化方法（Nagle 算法），将多次间隔 较小且数据量小的数据，合并成一个大的数据块，然后进行封包。这样做虽然提高了效率，但是接收端就难于 分辨出完整的数据包了，因为面向流的通信是无消息保护边界的

2) 由于 TCP 无消息保护边界, 需要在接收端处理消息边界问题，也就是我们所说的粘包、拆包问题, 看一张图

3) 示意图 TCP 粘包、拆包图解

![Xnip2020-04-05_14-07-22](/img/Xnip2020-04-05_14-07-22.png)

对图的说明:

假设客户端分别发送了两个数据包 D1 和 D2 给服务端，由于服务端一次读取到字节数是不确定的，故可能存在以 下四种情况： 

1) 服务端分两次读取到了两个独立的数据包，分别是 D1 和 D2，没有粘包和拆包

2) 服务端一次接受到了两个数据包，D1 和 D2 粘合在一起，称之为 TCP 粘包 

3) 服务端分两次读取到了数据包，第一次读取到了完整的 D1 包和 D2 包的部分内容，第二次读取到了 D2 包的剩余内容，这称之为 TCP 拆包

4) 服务端分两次读取到了数据包，第一次读取到了 D1 包的部分内容 D1_1，第二次读取到了 D1 包的剩余部 分内容 D1_2 和完整的 D2 包。

# TCP 粘包和拆包现象实例

## MyServer

```java
package com.nettycode.stickingandunpacking;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-04-03 16:49:26
 * @LastEditTime: 2020-04-03 16:49:43
 * @LastEditors: 麦子
 */


import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class MyServer {
    public static void main(String[] args) throws Exception{

        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {

            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup,workerGroup).channel(NioServerSocketChannel.class).childHandler(new MyServerInitializer()); //自定义一个初始化类


            ChannelFuture channelFuture = serverBootstrap.bind(7000).sync();
            channelFuture.channel().closeFuture().sync();

        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }

    }
}

```

## MyServerInitializer

```java
package com.nettycode.stickingandunpacking;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-04-03 16:49:26
 * @LastEditTime: 2020-04-03 16:50:06
 * @LastEditors: 麦子
 */



import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;


public class MyServerInitializer extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();

        pipeline.addLast(new MyServerHandler());
    }
}

```

## MyServerHandler

```java
package com.nettycode.stickingandunpacking;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

import java.nio.charset.Charset;
import java.util.UUID;

public class MyServerHandler extends SimpleChannelInboundHandler<ByteBuf>{
    private int count;

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //cause.printStackTrace();
        ctx.close();
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, ByteBuf msg) throws Exception {

        byte[] buffer = new byte[msg.readableBytes()];
        msg.readBytes(buffer);

        //将buffer转成字符串
        String message = new String(buffer, Charset.forName("utf-8"));

        System.out.println("服务器接收到数据 " + message);
        System.out.println("服务器接收到消息量=" + (++this.count));

        //服务器回送数据给客户端, 回送一个随机id ,
        ByteBuf responseByteBuf = Unpooled.copiedBuffer(UUID.randomUUID().toString() + " ", Charset.forName("utf-8"));
        ctx.writeAndFlush(responseByteBuf);

    }
}

```

## MyClient

```java
package com.nettycode.stickingandunpacking;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-04-03 16:49:26
 * @LastEditTime: 2020-04-03 16:49:50
 * @LastEditors: 麦子
 */



import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;

public class MyClient {
    public static void main(String[] args)  throws  Exception{

        EventLoopGroup group = new NioEventLoopGroup();

        try {

            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group).channel(NioSocketChannel.class)
                    .handler(new MyClientInitializer()); //自定义一个初始化类

            ChannelFuture channelFuture = bootstrap.connect("localhost", 7000).sync();

            channelFuture.channel().closeFuture().sync();

        }finally {
            group.shutdownGracefully();
        }
    }
}

```

## MyClientInitializer

```java
package com.nettycode.stickingandunpacking;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-04-03 16:49:26
 * @LastEditTime: 2020-04-03 16:49:58
 * @LastEditors: 麦子
 */

import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;


public class MyClientInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {

        ChannelPipeline pipeline = ch.pipeline();
        pipeline.addLast(new MyClientHandler());
    }
}

```

## MyClientHandler

```java
package com.nettycode.stickingandunpacking;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

import java.nio.charset.Charset;

public class MyClientHandler extends SimpleChannelInboundHandler<ByteBuf> {

    private int count;
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        //使用客户端发送10条数据 hello,server 编号
        for(int i= 0; i< 10; ++i) {
            ByteBuf buffer = Unpooled.copiedBuffer("hello,server " + i, Charset.forName("utf-8"));
            ctx.writeAndFlush(buffer);
        }
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, ByteBuf msg) throws Exception {
        byte[] buffer = new byte[msg.readableBytes()];
        msg.readBytes(buffer);

        String message = new String(buffer, Charset.forName("utf-8"));
        System.out.println("客户端接收到消息=" + message);
        System.out.println("客户端接收消息数量=" + (++this.count));

    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}

```

## 运行结果

```java
服务器接收到数据 hello,server 0hello,server 1hello,server 2hello,server 3hello,server 4hello,server 5hello,server 6hello,server 7hello,server 8hello,server 9
服务器接收到消息量=1



服务器接收到数据 hello,server 0
服务器接收到消息量=1
服务器接收到数据 hello,server 1hello,server 2hello,server 3
服务器接收到消息量=2
服务器接收到数据 hello,server 4hello,server 5hello,server 6
服务器接收到消息量=3
服务器接收到数据 hello,server 7hello,server 8hello,server 9
服务器接收到消息量=4
```

 可以看到， 这里都是明显的不确定数据按几次发送过去的。 



# TCP 粘包和拆包解决方案

1) 使用自定义协议 + 编解码器 来解决

2) 关键就是要解决 服务器端每次读取数据长度的问题, 这个问题解决，就不会出现服务器多读或少读数据的问 题，从而避免的 TCP 粘包、拆包 。

## 看一个具体的实例

1) 要求客户端发送 5 个 Message 对象, 客户端每次发送一个 Message 对象

2) 服务器端每次接收一个 Message, 分 5 次进行解码， 每读取到 一个 Message , 会回复一个 Message 对象 给客 户端.



## 代码实例

### MyServer

```java
package com.nettycode.stickyandunpackingsolutions;


import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class MyServer {
    public static void main(String[] args) throws Exception{

        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {

            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup,workerGroup).channel(NioServerSocketChannel.class).childHandler(new MyServerInitializer()); //自定义一个初始化类


            ChannelFuture channelFuture = serverBootstrap.bind(7000).sync();
            channelFuture.channel().closeFuture().sync();

        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }

    }
}

```

### MyServerInitializer

```java
package com.nettycode.stickyandunpackingsolutions;



import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;


public class MyServerInitializer extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();

        pipeline.addLast(new MyMessageDecoder());//解码器
        pipeline.addLast(new MyMessageEncoder());//编码器
        pipeline.addLast(new MyServerHandler());
    }
}

```

### MyServerHandler

```java
package com.nettycode.stickyandunpackingsolutions;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

import java.nio.charset.Charset;
import java.util.UUID;


//处理业务的handler
public class MyServerHandler extends SimpleChannelInboundHandler<MessageProtocol>{
    private int count;

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //cause.printStackTrace();
        ctx.close();
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, MessageProtocol msg) throws Exception {

        //接收到数据，并处理
        int len = msg.getLen();
        byte[] content = msg.getContent();

        System.out.println();
        System.out.println();
        System.out.println();
        System.out.println("服务器接收到信息如下");
        System.out.println("长度=" + len);
        System.out.println("内容=" + new String(content, Charset.forName("utf-8")));

        System.out.println("服务器接收到消息包数量=" + (++this.count));

        //回复消息

        String responseContent = UUID.randomUUID().toString();
        int responseLen = responseContent.getBytes("utf-8").length;
        byte[]  responseContent2 = responseContent.getBytes("utf-8");
        //构建一个协议包
        MessageProtocol messageProtocol = new MessageProtocol();
        messageProtocol.setLen(responseLen);
        messageProtocol.setContent(responseContent2);

        ctx.writeAndFlush(messageProtocol);


    }
}

```

### MessageProtocol

```java
package com.nettycode.stickyandunpackingsolutions;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-04-03 17:00:27
 * @LastEditTime: 2020-04-03 17:01:18
 * @LastEditors: 麦子
 */


//协议包
public class MessageProtocol {
    private int len; //关键
    private byte[] content;

    public int getLen() {
        return len;
    }

    public void setLen(int len) {
        this.len = len;
    }

    public byte[] getContent() {
        return content;
    }

    public void setContent(byte[] content) {
        this.content = content;
    }
}

```

### MyMessageEncoder

```java
package com.nettycode.stickyandunpackingsolutions;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.MessageToByteEncoder;

public class MyMessageEncoder extends MessageToByteEncoder<MessageProtocol> {
    @Override
    protected void encode(ChannelHandlerContext ctx, MessageProtocol msg, ByteBuf out) throws Exception {
        System.out.println("MyMessageEncoder encode 方法被调用");
        out.writeInt(msg.getLen());
        out.writeBytes(msg.getContent());
    }
}

```

### MyMessageDecoder

```java
package com.nettycode.stickyandunpackingsolutions;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.ReplayingDecoder;

import java.util.List;

public class MyMessageDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        System.out.println("MyMessageDecoder decode 被调用");
        //需要将得到二进制字节码-> MessageProtocol 数据包(对象)
        int length = in.readInt();

        byte[] content = new byte[length];
        in.readBytes(content);

        //封装成 MessageProtocol 对象，放入 out， 传递下一个handler业务处理
        MessageProtocol messageProtocol = new MessageProtocol();
        messageProtocol.setLen(length);
        messageProtocol.setContent(content);

        out.add(messageProtocol);

    }
}

```

### MyClient

```java
package com.nettycode.stickyandunpackingsolutions;



import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;

public class MyClient {
    public static void main(String[] args)  throws  Exception{

        EventLoopGroup group = new NioEventLoopGroup();

        try {

            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group).channel(NioSocketChannel.class)
                    .handler(new MyClientInitializer()); //自定义一个初始化类

            ChannelFuture channelFuture = bootstrap.connect("localhost", 7000).sync();

            channelFuture.channel().closeFuture().sync();

        }finally {
            group.shutdownGracefully();
        }
    }
}

```

### MyClientInitializer

```java
package com.nettycode.stickyandunpackingsolutions;

import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;


public class MyClientInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {

        ChannelPipeline pipeline = ch.pipeline();
        pipeline.addLast(new MyMessageEncoder()); //加入编码器
        pipeline.addLast(new MyMessageDecoder()); //加入解码器
        pipeline.addLast(new MyClientHandler());
    }
}

```

### MyClientHandler

```java
package com.nettycode.stickyandunpackingsolutions;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-04-03 17:00:26
 * @LastEditTime: 2020-04-05 15:51:43
 * @LastEditors: 麦子
 */

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

import java.nio.charset.Charset;

public class MyClientHandler extends SimpleChannelInboundHandler<MessageProtocol> {

    private int count;
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        //使用客户端发送10条数据 "今天天气冷，吃火锅" 编号

        for(int i = 0; i< 5; i++) {
            String mes = "今天天气冷，吃火锅";
            byte[] content = mes.getBytes(Charset.forName("utf-8"));
            int length = mes.getBytes(Charset.forName("utf-8")).length;

            //创建协议包对象
            MessageProtocol messageProtocol = new MessageProtocol();
            messageProtocol.setLen(length);
            messageProtocol.setContent(content);
            ctx.writeAndFlush(messageProtocol);

        }

    }

//    @Override
    protected void channelRead0(ChannelHandlerContext ctx, MessageProtocol msg) throws Exception {

        int len = msg.getLen();
        byte[] content = msg.getContent();

        System.out.println("客户端接收到消息如下");
        System.out.println("长度=" + len);
        System.out.println("内容=" + new String(content, Charset.forName("utf-8")));

        System.out.println("客户端接收消息数量=" + (++this.count));

    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        System.out.println("异常消息=" + cause.getMessage());
        ctx.close();
    }
}

```

## 运行效果

```java
服务器接收到信息如下
长度=27
内容=今天天气冷，吃火锅
服务器接收到消息包数量=1
MyMessageEncoder encode 方法被调用
MyMessageDecoder decode 被调用



服务器接收到信息如下
长度=27
内容=今天天气冷，吃火锅
服务器接收到消息包数量=2
MyMessageEncoder encode 方法被调用
MyMessageDecoder decode 被调用



服务器接收到信息如下
长度=27
内容=今天天气冷，吃火锅
服务器接收到消息包数量=3
MyMessageEncoder encode 方法被调用
MyMessageDecoder decode 被调用



服务器接收到信息如下
长度=27
内容=今天天气冷，吃火锅
服务器接收到消息包数量=4
MyMessageEncoder encode 方法被调用
MyMessageDecoder decode 被调用



服务器接收到信息如下
长度=27
内容=今天天气冷，吃火锅
服务器接收到消息包数量=5
MyMessageEncoder encode 方法被调用
MyMessageDecoder decode 被调用



服务器接收到信息如下
长度=27
内容=今天天气冷，吃火锅
服务器接收到消息包数量=1
MyMessageEncoder encode 方法被调用
MyMessageDecoder decode 被调用



服务器接收到信息如下
长度=27
内容=今天天气冷，吃火锅
服务器接收到消息包数量=2
MyMessageEncoder encode 方法被调用
MyMessageDecoder decode 被调用



服务器接收到信息如下
长度=27
内容=今天天气冷，吃火锅
服务器接收到消息包数量=3
MyMessageEncoder encode 方法被调用
MyMessageDecoder decode 被调用



服务器接收到信息如下
长度=27
内容=今天天气冷，吃火锅
服务器接收到消息包数量=4
MyMessageEncoder encode 方法被调用
MyMessageDecoder decode 被调用



服务器接收到信息如下
长度=27
内容=今天天气冷，吃火锅
服务器接收到消息包数量=5
MyMessageEncoder encode 方法被调用

```

可以看到每次都发送5个数据包。 其中的核心代码

```java
public class MyMessageEncoder extends MessageToByteEncoder<MessageProtocol> {
    @Override
    protected void encode(ChannelHandlerContext ctx, MessageProtocol msg, ByteBuf out) throws Exception {
        System.out.println("MyMessageEncoder encode 方法被调用");
        out.writeInt(msg.getLen());
        out.writeBytes(msg.getContent());
    }
}
```

每次发送的时候记录了长度length。
---
title:       "Netty编解码器和handler的调用机制"
subtitle:    ""
description: "出站入站说明，组件之间的关系和作用，编码解码器"
date:        2020-04-03
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["网络编程"]
categories:  ["Tech" ]
---

[TOC]

**说明文章资料来源：尚硅谷韩顺平Netty视频教程（2019发布） https://www.bilibili.com/video/av76227904?p=36**

# 基本说明

1) netty 的组件设计：Netty 的主要组件有 Channel、EventLoop、ChannelFuture、ChannelHandler、ChannelPipe 等

2) ChannelHandler 充当了处理入站和出站数据的应用程序逻辑的容器。例如，实现 ChannelInboundHandler 接口（或 ChannelInboundHandlerAdapter），你就可以接收入站事件和数据，这些数据会被业务逻辑处理。当要给客户端 发 送 响 应 时 ， 也 可 以 从 ChannelInboundHandler 冲 刷 数 据 。 业 务 逻 辑 通 常 写 在 一 个 或 者 多 个 ChannelInboundHandler 中。ChannelOutboundHandler 原理一样，只不过它是用来处理出站数据的

3) **ChannelPipeline 提供了 ChannelHandler 链的容器**。以客户端应用程序为例，如果事件的运动方向是从客户端到 服务端的， **那么我们称这些事件为出站的**， 即客户端发送给服务端的数据会通过 pipeline 中的一系列 ChannelOutboundHandler，并被这些 Handler 处理，**反之则称为入站的**

![Xnip2020-04-03_15-06-50](/img/Xnip2020-04-03_15-06-50.png)

# 编码解码器

1) 当 Netty 发送或者接受一个消息的时候，就将会发生一次数据转换。入站消息会被解码：从字节转换为另一种 格式（比如 java 对象）；如果是出站消息，它会被编码成字节。

2) Netty 提供一系列实用的编解码器，**他们都实现了 ChannelInboundHadnler 或者 ChannelOutboundHandler 接口。 在这些类中，channelRead 方法已经被重写了**。以入站为例，对于每个从入站 Channel 读取的消息，这个方法会 被调用。随后，它将调用由解码器所提供的 decode()方法进行解码，并将已经解码的字节转发给 ChannelPipeline 中的下一个 ChannelInboundHandler。



## 解码器-ByteToMessageDecoder

1) 关系继承图

![Xnip2020-04-03_15-26-44](/img/Xnip2020-04-03_15-26-44.png)

可以看到他也是一个入站的ChannelInboundHandler，所以可以放到ChannelPipeline容器中。 

```java
// 获取到 pipeline
ChannelPipeline pipeline = ch.pipeline();
// 向 pipeline 加入解码器
pipeline.addLast("decoder", new StringDecoder());
// 向 pipeline 加入编码器
pipeline.addLast("encoder", new StringEncoder());
// 加入自己的业务处理 handler
pipeline.addLast(new GroupChatServerHandler());
```



2) 由于不可能知道远程节点是否会一次性发送一个完整的信息，tcp 有可能出现粘包拆包的问题，这个类会对入 站数据进行缓冲，直到它准备好被处理.



3) 一个关于 ByteToMessageDecoder 实例分析

![Xnip2020-04-03_15-29-48](/img/Xnip2020-04-03_15-29-48.png)



## Netty 的 handler 链的调用机制

实例要求:

1) 使用自定义的编码器和解码器来说明 Netty 的 handler 调用机制 

​     客户端发送 long -> 服务器 

​     服务端发送 long -> 客户端

2) 案例演示

![Xnip2020-04-03_15-35-11](/img/Xnip2020-04-03_15-35-11.png)

**说明**

可以看到ChannelPipeline这个容器里面都是说有的入站和出站的所有的Handler，也就是所有的业务逻辑进行**队列形式**的排序。 



3) 结论

不论解码器 handler 还是 编码器 handler 即接收的消息类型必须与待处理的消息类型一致，否则该 handler 不 会被执行

在解码器 进行数据解码时，需要判断 缓存区(ByteBuf)的数据是否足够 ，否则接收到的结果会期望结果可能 不一致



## 解码器-ReplayingDecoder

1) public abstract class ReplayingDecoder<S> extends ByteToMessageDecoder

2) ReplayingDecoder 扩展了 ByteToMessageDecoder 类，使用这个类，我们不必调用 readableBytes()方法。参数 S 指定了用户状态管理的类型，其中 Void 代表不需要状态管理

3) 应用实例：使用 ReplayingDecoder 编写解码器，对前面的案例进行简化 [案例演示]

```java
import java.util.List;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.ReplayingDecoder;

/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-03-20 14:18:09
 * @LastEditTime: 2020-04-03 15:54:01
 * @LastEditors: 麦子
 */
public class App extends ReplayingDecoder<Void> {

    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        System.out.println("MyByteToLongDecoder2 被调用"); 
        //在 ReplayingDecoder 不需要判断数据是否足够读取，内部会进行处理判断
        out.add(in.readLong());
    }
 
}
```

4) ReplayingDecoder 使用方便，但它也有一些局限性：

1. 并 不 是 所 有 的 ByteBuf 操 作 都 被 支 持 ， 如 果 调 用 了 一 个 不 被 支 持 的 方 法 ， 将 会 抛 出 一 个 UnsupportedOperationException。

2. ReplayingDecoder 在某些情况下可能稍慢于 ByteToMessageDecoder， 例如网络缓慢并且消息格式复杂时， 消息会被拆成了多个碎片，速度变慢

   

## 其它编解码器

1) LineBasedFrameDecoder：这个类在 Netty 内部也有使用，它使用行尾控制字符（\n 或者\r\n）作为分隔符来解 析数据。

2) DelimiterBasedFrameDecoder：使用自定义的特殊字符作为消息的分隔符。

3) HttpObjectDecoder：一个 HTTP 数据的解码器

4) LengthFieldBasedFrameDecoder：通过指定长度来标识整包消息， 这样就可以自动的处理黏包和半包消息。

![Xnip2020-04-03_15-56-20](/img/Xnip2020-04-03_15-56-20.png)












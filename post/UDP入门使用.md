---
title:       "UDP入门使用"
subtitle:    ""
description: ""
date:        2020-01-08
author:      "麦子"
image:       "https://get.pxhere.com/photo/snow-macro-snowflake-black-darkness-phenomenon-computer-wallpaper-screenshot-black-and-white-font-grass-night-midnight-1420032.jpg"
tags:        ["网络编程"]
categories:  ["Tech" ]
---

[TOC]

# 概述

转载地址：https://segmentfault.com/a/1190000008543293

UDP 全称 User Datagram Protocol, 与 TCP 同是在网络模型中的传输层的协议。

UDP是面向无连接的通信，它发送的是数据包，效率高，但是他不保证通信的可靠，也就是说他不保证数据包能完全到达目的主机。

![Xnip2020-01-08_12-36-19](/img/Xnip2020-01-08_12-36-19.png)

# 主要特点

- [ ] **无连接的**，即发送数据之前不需要建立连接，因此减少了开销和发送数据之前的时延。
- [ ] **不保证可靠交付**，因此主机不需要为此复杂的连接状态表
- [ ] **面向报文的**，意思是 UDP 对应用层交下来的报文，既不合并，也不拆分，而是保留这些报文的边界，在添加首部后向下交给 IP 层。
- [ ] **没有阻塞控制**，因此网络出现的拥塞不会使发送方的发送速率降低。
- [ ] **支持一对一、一对多、多对一和多对多的交互通信**，也即是提供广播和多播的功能。
- [ ] **首部开销小**，首部只有 8 个字节，分为四部分。

# 常用场景

UDP 的速度很快。在 **需要不建立连接即可发送数据** 的系统，或者 **保证最快的传输速度比每一位数据都正确更重要** 的系统（如视频会议，丢失某个数据包只是一个画面或者声音的小干扰）中，UDP 才是正确的选择。实际上，在同一个网段，或者在信号很好的局域网，UDP 是非常可靠的。

1. 名字转换（DNS）
2. 文件传送（TFTP）
3. 路由选择协议（RIP）
4. IP 地址配置（BOOTP，DHTP）
5. 网络管理（SNMP）
6. 远程文件服务（NFS）
7. IP 电话
8. 流式多媒体通信

# 报文结构

UDP 数据报分为数据字段和首部字段。

首部字段只有 8 个字节，由四个字段组成，每个字段的长度是 2 个字节。

![Xnip2020-01-11_11-10-08](/img/Xnip2020-01-11_11-10-08.png)

**首部各字段意义**：

**源端口**：源端口号，在需要对方回信时选用，不需要时可全 0.

**目的端口**：目的端口号，在终点交付报文时必须要使用到。

**长度**：UDP 用户数据报的长度，在只有首部的情况，其最小值是 8 。

**检验和**：检测 UDP 用户数据报在传输中是否有错，有错就丢弃。



### 数据包大小

转载地址：https://blog.51cto.com/10324228/1983469

**在链路层**，由以太网的物理特性决定了数据帧的长度为（46＋18）---（1500＋18），其中的18是链路层的首部和尾部18Bytes，也就是说数据帧的内容最大为1500（不包括帧头和帧尾），事实上，这个1500就是网络层的IP数据报的长度限制，即MTU（Maximum Transmission Unit）为1500； 　

**在网络层**，因为IP包的首部要占用20字节，所以这的MTU为1500－20＝1480，这个1480就是用来存放TCP传来的TCP报文段或者UDP传来的UDP数据报的；

**在传输层**，对于UDP包的首部要占用8字节，所以这的MTU为1480－8＝1472，也就是用户可以使用的部分；

**UDP 包的大小就应该是 1500 - IP头(20) - UDP头(8) = 1472(Bytes)**
**TCP 包的大小就应该是 1500 - IP头(20) - TCP头(20) = 1460 (Bytes)**

如果我们定义的TCP和UDP包没有超过范围，那么我们的包在IP层就不用分包了，这样传输过程中就避免了在IP层组包发生的错误；如果超过范围，既IP数据报大于1500字节，发送方IP层就需要将数据包分成若干片，而接收方IP层就需要进行数据报的重组。更严重的是，如果使用UDP协议，当IP层组包发生错误，那么包就会被丢弃。接收方无法重组数据报，将导致丢弃整个IP数据报。UDP不保证可靠传输；但是TCP发生组包错误时，该包会被重传，保证可靠传输。

UDP数据报的长度是指包括报头和数据部分在内的总字节数，其中报头长度固定，数据部分可变。数据报的最大长度根据操作环境的不同而各异。**从理论上说，包含报头在内的数据报的最大长度为65535字节(64K)，我们在用Socket编程时，UDP协议要求包小于64K。**

### 实际应用

用UDP协议发送时，用sendto函数最大能发送数据的长度为：65535- IP头(20) - UDP头(8)＝65507字节。用sendto函数发送数据时，如果发送数据长度大于该值，则函数会返回错误。  

用TCP协议发送时，由于TCP是数据流协议，因此不存在包大小的限制（暂不考虑缓冲区的大小），这是指在用send函数时，数据长度参数不受限制。而实际上，所指定的这段数据并不一定会一次性发送出去，如果这段数据比较长，会被分段发送，如果比较短，可能会等待和下一次数据一起发送。

# 核心API

转载地址：https://segmentfault.com/a/1190000007599375

在 Java 中使用 UDP 时关键的两个类分别是：**DatagramSocket** 和 **DatagramPacket**。DatagramPacket 表示数据包，而 DatagrameSocket 用来发送和接收数据包。

因为 UDP 协议并不需要建立连接，所以我们将数据（byte数组）放入 DatagramPacket 之后，还需要将目的地（IP地址和端口）放入到 DatagramPacket 中 —— DatagramSocket 的 *send(DatagramPacket packet)* 方法根据 packet 中指定目的地，将其包含的数据往这个目的地发送。至于数据是否能（准确）到达目的地，DatagramSocket 并不关心。

### DatagramPacket

原文链接：https://blog.csdn.net/Robot__Man/article/details/80657727

#### 构建接收包

　- DatagramPacket(byte[] buf,int length)，将数据包中length长的数据装进buf数组。
　　- DatagramPacket(byte[] buf,int offset,int length)，将数据包中从offset开始，length长的数据装进buf数组。


#### 构建发送包

- -DatagramPacket(byte[] buf,int length,InetAddress clientAddress,int clientPort)，从buf数组中，取出length长的数据创建数据包对象，目标是clientAddress地址，clientPort端口，通常用来发送数据给客户端。
- -DatagramPacket(byte[] buf,int offset,int length,InetAddress clientAddress,int clientPort)，从buf数组中，取出offset开始的、length长的数据创建数据包对象，目标是clientAddress地址，clientPort端口，通常用来发送数据给客户端。


### DatagramSocket

DatagramSocket 在接收数据包时，我们需要为其指定一个监听的端口。当有包含了接收端机器 IP 地址和 DatagramSocket 所监听端口的数据包到达时，DatagramSocket 的 *receive(DatagramPacket packet)* 方法便会对数据包进行接收，并将接收到的数据包填入到 packet 中。

#### 服务端接收

- DatagramSocket(int port)，创建实例，并固定监听port端口的报文，通常用于服务端。
　　- receive(DatagramPacket d)，接收数据报文到d中，receive方法产生“阻塞”。

#### 客户端发送

无参的构造方法DatagramSocket()，通常用于客户端编程，它并没有特定监听的端口，仅仅使用一个临时的。程序会让操作系统分配一个可用的端口。

send(DatagramPacket dp)，该方法用于发送报文dp到目的地。



## 注意

现在让我们来实践 DatagramSocket 和 DatagramPacket：

因为 UDP 没有服务端和客户端之分，所以我们把两端分别定义为 发送端 和 接收端。



## 单播代码演示

### 发送端代码

```java
public class UDPSender {
    public static void main(String[] args) {
        try {
            DatagramSocket ds = new DatagramSocket();
            String msgBody = "Hello word ";
            DatagramPacket dp = new DatagramPacket(msgBody.getBytes(), 0, msgBody.length(),
                    InetAddress.getByName("127.0.0.1"), 3000);
            ds.send(dp);
            ds.close();
            System.out.println("发送完成");
        } catch (SocketException e) {
            e.printStackTrace();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 接收端

```java
public class UDPReceiver {
    public static void main(String[] args) {
        try {
            DatagramSocket ds = new DatagramSocket(3000);
            byte[] buf = new byte[1024];
            DatagramPacket dp = new DatagramPacket(buf, buf.length);

            ds.receive(dp);
            System.out.println(dp.getData().length);
            System.out.println(dp.getSocketAddress());
            System.out.println(ds.getInetAddress());
            String data = " form :" + dp.getAddress().getHostAddress()+" "+ new String(dp.getData(), 0, dp.getLength());
            System.out.println(data);

        } catch (SocketException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 运行说明

需要先启动接收端， 因为

```java
 ds.receive(dp);
```

这个代码是要监听数据是否来了， 如果没来就是一直阻塞中的。TCP中的

```java
 Socket client = server.accept();
```

是一样的。 



# UDP信息传递的方式分

原文链接：https://blog.csdn.net/ljheee/article/details/51722792

## 单播

是客户端与服务器之间的点到点连接。

## 多播

主机之间“一对一组”的通讯模式，也就是加入了同一个组的主机可以接受到此组内的所有数据。

### 组播

组播是一对多的传输方式，其中有个组播组的 概念，发送端将数据向一个组内发送，网络中的路由器通过底层的IGMP协议自动将数据发送到所有监听这个组的终端。至于广播则和组播有一些相似，区别是路由器向子网内的每一个终端都投递一份数据包，不论这些终端是否乐于接收该数据包。**UDP广播只能在内网（同一网段）有效，而组播可以较好实现跨网段群发数据。**

### 组播的原理

组播首先由一个用户申请一个组播组，这个组播组被维护在路由器中，其他用户申请加入组播组，这样当一个用户向组内发送消息时，路由器将消息转发给组内的所有成员。如果申请加入的组不在本级路由中，如果路由器和交换机允许组播协议通过，路由器将申请加入的操作向上级路由提交。广域网通信要经过多级路由器和交换机，几乎所有的网络设备都默认阻止组播协议通过(只允许本网段内，不向上级提交)，这使得广域网上实现组播有一定局限。

### UDP组播的基本步骤

1. 建立socket
2. socket和端口绑定
3. 加入一个组播组
4. 通过sendto / recvfrom进行数据的收发
5. 关闭socket

 **服务器和客户端必须都要加入相同的组播地址才可以。**

### 实际操作

多播数据报套接字类用于发送和接收 IP 多播包。MulticastSocket 是一种 (UDP) DatagramSocket，它具有加入 Internet 上其他多播主机的“组”的附加功能。

多播组通过 D 类 IP 地址和标准 UDP 端口号指定。可以通过首先使用所需端口创建 MulticastSocket，然后调用 joinGroup(InetAddress groupAddr) 方法来加入多播组。

在JAVA中，多播一样十分好实现，要实现多播，就要用到MulticastSocket类，其实该类就是DatagramSocket的子类，在使用时除了多播自己的一些特性外，把它当做DatagramSocket类使用就可以了。



## 广播

主机之间“一对所有”的通讯模式，广播者可以向网络中所有主机发送信息。**广播禁止在Internet宽带网上传输**（广播风暴）。



## 注意

只有UDP才有广播、组播的传递方式；而TCP是一对一连接通信。多播的重点是高效的把同一个包尽可能多的发送到不同的，甚至可能是未知的设备。但是TCP连接是一对一明确的，只能单播。

# udp为什么会丢失数据

转载地址： https://www.cnblogs.com/mengyan/archive/2012/10/04/2711340.html

## 主要丢包原因

1、接收端处理时间过长导致丢包：调用recv方法接收端收到数据后，处理数据花了一些时间，处理完后再次调用recv方法，在这二次调用间隔里,发过来的包可能丢失。对于这种情况可以修改接收端，将包接收后存入一个缓冲区，然后迅速返回继续recv。

2、发送的包巨大丢包：虽然send方法会帮你做大包切割成小包发送的事情，但包太大也不行。例如超过50K的一个udp包，不切割直接通过send方法发送也会导致这个包丢失。这种情况需要切割成小包再逐个send。

3、发送的包较大，超过接受者缓存导致丢包：包超过mtu size数倍，几个大的udp包可能会超过接收者的缓冲，导致丢包。这种情况可以设置socket接收缓冲。以前遇到过这种问题，我把接收缓冲设置成64K就解决了。

4、发送的包频率太快：虽然每个包的大小都小于mtu size 但是频率太快，例如40多个mut size的包连续发送中间不sleep，也有可能导致丢包。这种情况也有时可以通过设置socket接收缓冲解决，但有时解决不了。所以在发送频率过快的时候还是考虑sleep一下吧。

5、局域网内不丢包，公网上丢包。这个问题我也是通过切割小包并sleep发送解决的。如果流量太大，这个办法也不灵了。总之udp丢包总是会有的，如果出现了用我的方法解决不了，还有这个几个方法： 要么减小流量，要么换tcp协议传输，要么做丢包重传的工作。



## 具体问题分析

### 1.   发送频率过高导致丢包

很多人会不理解发送速度过快为什么会产生丢包，原因就是UDP的SendTo不会造成线程阻塞，也就是说，UDP的SentTo不会像TCP中的SendTo那样，直到数据完全发送才会return回调用函数，它不保证当执行下一条语句时数据是否被发送。（SendTo方法是异步的）这样，如果要发送的数据过多或者过大，那么在缓冲区满的那个瞬间要发送的报文就很有可能被丢失。至于对“过快”的解释，作者这样说：“A few packets a second are not an issue; hundreds or thousands may be an issue.”（一秒钟几个数据包不算什么，但是一秒钟成百上千的数据包就不好办了）。 要解决接收方丢包的问题很简单，首先要保证程序执行后马上开始监听（如果数据包不确定什么时候发过来的话），其次，要在收到一个数据包后最短的时间内重新回到监听状态，其间要尽量避免复杂的操作（比较好的解决办法是使用多线程回调机制）。

### 2.  报文过大丢包

至于报文过大的问题，可以通过控制报文大小来解决，使得每个报文的长度小于MTU。以太网的MTU通常是1500 bytes，其他一些诸如拨号连接的网络MTU值为1280 bytes，如果使用speaking这样很难得到MTU的网络，那么最好将报文长度控制在1280 bytes以下。

### 3.  发送方丢包

发送方丢包：内部缓冲区（internal buffers）已满，并且发送速度过快（即发送两个报文之间的间隔过短）；  接收方丢包：Socket未开始监听；  虽然UDP的报文长度最大可以达到64 kb，但是当报文过大时，稳定性会大大减弱。这是因为当报文过大时会被分割，使得每个分割块（翻译可能有误差，原文是fragmentation）的长度小于MTU，然后分别发送，并在接收方重新组合（reassemble），但是如果其中一个报文丢失，那么其他已收到的报文都无法返回给程序，也就无法得到完整的数据了。 

### 4.  设置系统默认缓存区大小

### 5.  采用多线程方式接收数据，将接收和处理数据过程分开

# 组播代码演示

虚拟机设置

```java
"vmArgs":"-Djava.net.preferIPv4Stack=true"
```

### 发送端

```java
public class MulticastUDPSender {
    public static void main(String[] args) throws Exception {
        int mcPort = 12345;
        String mcIPStr = "230.0.0.0";
        DatagramSocket udpSocket = new DatagramSocket();

        InetAddress mcIPAddress = InetAddress.getByName(mcIPStr);
        byte[] msg = "Hello maizi 007".getBytes();
        DatagramPacket packet = new DatagramPacket(msg, msg.length);
        packet.setAddress(mcIPAddress);
        packet.setPort(mcPort);
        udpSocket.send(packet);

        System.out.println("Sent a  multicast message.");
        System.out.println("Exiting application");
        udpSocket.close();
    }
}
```

### 接收端

```java
public class MulticastUDPReceiver {
    public static void main(String[] args) throws Exception {
        // System.setProperty("java.net.preferIPv4Stack", "true");
        
        int mcPort = 12345;
        String mcIPStr = "230.0.0.0";
        MulticastSocket mcSocket = null;
        InetAddress mcIPAddress = null;
        mcIPAddress = InetAddress.getByName(mcIPStr);
        mcSocket = new MulticastSocket(mcPort);
        System.out.println("Multicast Receiver running at:" + mcSocket.getLocalSocketAddress());
        mcSocket.joinGroup(mcIPAddress);

        DatagramPacket packet = new DatagramPacket(new byte[1024], 1024);

        System.out.println("Waiting for a  multicast message...");
        mcSocket.receive(packet);
        String msg = new String(packet.getData(), packet.getOffset(), packet.getLength());
        System.out.println("[Multicast  Receiver] Received:" + msg);

        mcSocket.leaveGroup(mcIPAddress);
        mcSocket.close();
    }
}
```

### 运行结果

```java
Multicast Receiver running at:0.0.0.0/0.0.0.0:12345
Waiting for a  multicast message...
[Multicast  Receiver] Received:Hello maizi 007
```

# 总结

虽然 UDP 是不可靠的协议，但是因为它不需要建立连接，效率更快，所以在 需要不建立连接即可发送数据 的系统（比如本文的例子），或者 保证最快的传输速度比每一位数据都正确更重要 的系统中，我们应该使用 UDP。当然，基于 UDP 我们同样可以开发出可靠的协议——数据包的正确与否可以交给应用程序来判断，如果有问题接收端便提示发送端重新发送。
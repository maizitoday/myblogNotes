---
title:       "UDP辅助TCP实现点对点数据传输"
subtitle:    ""
description: ""
date:        2020-01-11
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["网络编程"]
categories:  ["Tech" ]
---

[TOC]

**转载地址:   https://blog.csdn.net/weixin_42089175/article/details/88697374**

# 大致流程

先启动一个UDP的接收端，监听端口**30201**， 再启动一个UDP的发送端，进行广播发送到**30201**，同时这个发送端也进行一个监听端口**30202**的UDP线程接受。这个**30202**会作为参数发送给接收端UDP。

当UDP发送端发送给UDP的接受端的时候，接收端马上也向这个原来发送端**30202**的UDP回复一条消息，这个消息里面就带有TCP的连接的端口**30401**

我们把接受UDP端的程序中启动**ServerSocket**，在UDP的发送端启动**Socket**，如此端口就已经知道了，也就完成了通过UDP知道接受广播的IP地址和端口，从而进行TCP网络的连接。 

**注意： ** 我们把需要的口令和参数通过map封装，然后转成byte进行传输。

# 具体代码如下



## 客户端代码

### Client

```java
import java.io.IOException;

import com.tcp.udp.client.bean.ServerInfo;

public class Client {
    public static void main(String[] args) {
        ServerInfo info = UDPSearcher.searchServer(10000);
        System.out.println("Server:" + info);

        if (info != null) {
            try {
                TCPClient.linkWith(info );
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```



### UDPSearcher

udp的发送和监听，最后获取TCP服务端的IP和端口

```java
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.nio.ByteBuffer;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

import com.tcp.udp.client.bean.ServerInfo;
import com.tcp.udp.clink.utils.ByteUtils;
import com.tcp.udp.clink.utils.JsonSerilizable;
import com.tcp.udp.constants.UDPConstants;

public class UDPSearcher {
    private static final int LISTEN_PORT = UDPConstants.PORT_CLIENT_RESPONSE;

    public static ServerInfo searchServer(int timeout) {
        System.out.println("UDPSearcher Started.");

        // 成功收到回送的栅栏
        CountDownLatch receiveLatch = new CountDownLatch(1);
        Listener listener = null;
        try {
            // 发送广播
            listener = listen(receiveLatch);
            // 搜索10秒
            sendBroadcast();
            receiveLatch.await(timeout, TimeUnit.MILLISECONDS);
        } catch (Exception e) {
            e.printStackTrace();
        }
        // 完成
        System.out.println("UDPSearcher Finished.");
        if (listener == null) {
            return null;
        }
        List<ServerInfo> devices = listener.getServerAndClose();
        if (devices.size() > 0) {
            return devices.get(0);
        }
        return null;
    }

    private static Listener listen(CountDownLatch receiveLatch) throws InterruptedException {
        System.out.println("UDPSearcher start listen.");
        CountDownLatch startDownLatch = new CountDownLatch(1);
        Listener listener = new Listener(LISTEN_PORT, startDownLatch, receiveLatch);
        listener.start();
        startDownLatch.await();
        return listener;
    }

    private static void sendBroadcast() throws IOException {
        System.out.println("UDPSearcher sendBroadcast started.");

        // 作为搜索方，让系统自动分配端口
        DatagramSocket ds = new DatagramSocket();
        Map<String, Object> map = new HashMap<String, Object>();
        // CMD命名
        map.put("CMD", UDPConstants.UDP_PASSWORD_KEY);
        // 回送端口信息
        map.put("port", LISTEN_PORT);
        byte[] bytes = JsonSerilizable.changeMapToByte(map);

        // 直接构建packet
        DatagramPacket requestPacket = new DatagramPacket(bytes, bytes.length);
        // 广播地址
        requestPacket.setAddress(InetAddress.getByName("255.255.255.255"));
        // 设置服务器端口
        requestPacket.setPort(UDPConstants.PORT_SERVER);

        // 发送
        ds.send(requestPacket);
        ds.close();

        // 完成
        System.out.println("UDPSearcher sendBroadcast finished.");
    }

    private static class Listener extends Thread {
        private final int listenPort;
        private final CountDownLatch startDownLatch;
        private final CountDownLatch receiveDownLatch;
        private final List<ServerInfo> serverInfoList = new ArrayList<>();
        private final byte[] buffer = new byte[128];
        private boolean done = false;
        private DatagramSocket ds = null;

        private Listener(int listenPort, CountDownLatch startDownLatch, CountDownLatch receiveDownLatch) {
            super();
            this.listenPort = listenPort;
            this.startDownLatch = startDownLatch;
            this.receiveDownLatch = receiveDownLatch;
        }

        @Override
        public void run() {
            super.run();
            // 通知已启动
            startDownLatch.countDown();

            try {
                // 监听30202回送端口
                ds = new DatagramSocket(listenPort);
                // 构建接收实体
                DatagramPacket receivePack = new DatagramPacket(buffer, buffer.length);

                while (!done) {
                    // 接收
                    ds.receive(receivePack);

                    // 打印接收到的信息与发送者的信息
                    // 发送者的IP地址
                    String ip = receivePack.getAddress().getHostAddress();
                    byte[] data = receivePack.getData();

                    Map<String, Object> map = JsonSerilizable.changeByteToMap(data);
                    Iterator<Entry<String, Object>> entries = map.entrySet().iterator();

                    String cmdVal = null;
                    String snVal = null;
                    Integer serverPort = null;

                    while (entries.hasNext()) {
                        Map.Entry<String, Object> entry = entries.next();
                        String key = entry.getKey();
                        Object value = entry.getValue();
                        if (key.equals("CMD")) {
                            cmdVal = (String) value;
                        } else if (key.equals("sn")) {
                            snVal = (String) value;
                        } else if (key.equals("port")) {
                            serverPort = (Integer) value;
                        }
                    }
                    if (UDPConstants.UDP_PASSWORD_KEY.equals(cmdVal)) {
                        ServerInfo info = new ServerInfo(serverPort, ip, snVal);
                        serverInfoList.add(info);
                    }
                    // 成功接收到一份
                    receiveDownLatch.countDown();
                    done = true;
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                close();
            }
            System.out.println("UDPSearcher listener finished.");
        }

        private void close() {
            if (ds != null) {
                ds.close();
                ds = null;
                done = true;
            }
        }

        List<ServerInfo> getServerAndClose() {
            done = true;
            close();
            return serverInfoList;
        }
    }
}

```



### TCPClient

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.PrintStream;
import java.net.Inet4Address;
import java.net.InetSocketAddress;
import java.net.Socket;

import com.tcp.udp.client.bean.ServerInfo;

public class TCPClient {
    public static void linkWith(ServerInfo info) throws IOException {
        Socket socket = new Socket();
        // 超时时间
        socket.setSoTimeout(3000);

        // 连接本地，端口2000；超时时间3000ms
        socket.connect(new InetSocketAddress(Inet4Address.getByName(info.getAddress()), info.getPort()), 3000);

        System.out.println("已发起服务器连接，并进入后续流程～");
        System.out.println("客户端信息：" + socket.getLocalAddress() + " P:" + socket.getLocalPort());
        System.out.println("服务器信息：" + socket.getInetAddress() + " P:" + socket.getPort());

        try {
            // 发送接收数据
            todo(socket);
        } catch (Exception e) {
            System.out.println("异常关闭");
        }

        // 释放资源
        socket.close();
        System.out.println("客户端已退出～");

    }

    private static void todo(Socket client) throws IOException {
        // 构建键盘输入流
        InputStream in = System.in;
        BufferedReader input = new BufferedReader(new InputStreamReader(in));


        // 得到Socket输出流，并转换为打印流
        OutputStream outputStream = client.getOutputStream();
        PrintStream socketPrintStream = new PrintStream(outputStream);


        // 得到Socket输入流，并转换为BufferedReader
        InputStream inputStream = client.getInputStream();
        BufferedReader socketBufferedReader = new BufferedReader(new InputStreamReader(inputStream));

        boolean flag = true;
        do {
            // 键盘读取一行
            String str = input.readLine();
            // 发送到服务器
            socketPrintStream.println(str);


            // 从服务器读取一行
            String echo = socketBufferedReader.readLine();
            if ("bye".equalsIgnoreCase(echo)) {
                flag = false;
            } else {
                System.out.println(echo);
            }
        } while (flag);

        // 资源释放
        socketPrintStream.close();
        socketBufferedReader.close();

    }

}

```



## 服务端代码

### Server

```java
import java.io.IOException;

import com.tcp.udp.constants.TCPConstants;

public class Server {
    public static void main(String[] args) {
        TCPServer tcpServer = new TCPServer(TCPConstants.PORT_SERVER);
        boolean isSucceed = tcpServer.start();
        if (!isSucceed) {
            System.out.println("Start TCP server failed!");
            return;
        }

        UDPProvider.start(TCPConstants.PORT_SERVER);

        try {
            //noinspection ResultOfMethodCallIgnored
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }

        UDPProvider.stop();
        tcpServer.stop();
    }
}

```



### UDPProvider

udp的广播的接受和回复，解析对应的口令处理，把TCP服务启动的端口通过UDP发送给客户端。

```java
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.nio.ByteBuffer;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.Map.Entry;
import java.util.UUID;

import com.tcp.udp.clink.utils.JsonSerilizable;
import com.tcp.udp.constants.UDPConstants;

class UDPProvider {
    private static Provider PROVIDER_INSTANCE;

    static void start(int port) {
        stop();
        String sn = UUID.randomUUID().toString();
        Provider provider = new Provider(sn, port);
        provider.start();
        PROVIDER_INSTANCE = provider;
    }

    static void stop() {
        if (PROVIDER_INSTANCE != null) {
            PROVIDER_INSTANCE.exit();
            PROVIDER_INSTANCE = null;
        }
    }

    private static class Provider extends Thread {
        private final String sn;
        private final int port;
        private boolean done = false;
        private DatagramSocket ds = null;
        // 存储消息的Buffer
        final byte[] buffer = new byte[128];

        Provider(String sn, int port) {
            super();
            this.sn = sn;
            this.port = port;
        }

        @Override
        public void run() {
            super.run();
            System.out.println("UDPProvider Started.");
            try {
                // 监听30201 端口
                ds = new DatagramSocket(UDPConstants.PORT_SERVER);
                // 接收消息的Packet
                DatagramPacket receivePack = new DatagramPacket(buffer, buffer.length);
                while (!done) {
                    // 接收
                    ds.receive(receivePack);
                    // 打印接收到的信息与发送者的信息
                    // 发送者的IP地址
                    String clientIp = receivePack.getAddress().getHostAddress();
                    int clientPort = receivePack.getPort();
                    System.out.println(clientPort);
                    byte[] clientData = receivePack.getData();

                    Map<String, Object> map = JsonSerilizable.changeByteToMap(clientData);
                    Iterator<Entry<String, Object>> entries = map.entrySet().iterator();

                    String cmdVal = null;
                    int udpPort = 0;

                    while (entries.hasNext()) {
                        Map.Entry<String, Object> entry = entries.next();
                        String key = entry.getKey();
                        Object value = entry.getValue();
                        if (key.equals("CMD")) {
                            cmdVal = (String) value;
                        } else if (key.equals("port")) {
                            udpPort = (int) value;
                        }
                    }
                    // 判断合法性
                    if (UDPConstants.UDP_PASSWORD_KEY.equals(cmdVal)) {
                        // 构建一份回送数据
                        Map<String, Object> responseMap = new HashMap<String, Object>();
                        // CMD命名
                        responseMap.put("sn", sn);
                        // 回送端口信息
                        responseMap.put("port", port);
                        responseMap.put("CMD", UDPConstants.UDP_PASSWORD_KEY);

                        byte[] responseBytes = JsonSerilizable.changeMapToByte(responseMap);

                        // 直接根据发送者构建一份回送信息
                        DatagramPacket responsePacket = new DatagramPacket(responseBytes, responseBytes.length,
                                receivePack.getAddress(), udpPort);
                        ds.send(responsePacket);
                        System.out.println("UDPProvider response to:" + clientIp + "\tport:"
                                + udpPort + "\tdataLen:" + responseBytes.length);
                    } else {
                        System.out.println("UDPProvider receive cmd nonsupport; cmd:" + cmdVal + "\tport:" + port);
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                close();
            }
            // 完成
            System.out.println("UDPProvider Finished.");
        }

        private void close() {
            if (ds != null) {
                ds.close();
                ds = null;
            }
        }

        /**
         * 提供结束
         */
        void exit() {
            done = true;
            close();
        }
    }
}

```



### TCPServer

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.net.ServerSocket;
import java.net.Socket;

public class TCPServer {
    private final int port;
    private ClientListener mListener;

    public TCPServer(int port) {
        this.port = port;
    }

    public boolean start() {
        try {
            ClientListener listener = new ClientListener(port);
            mListener = listener;
            listener.start();
        } catch (IOException e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }

    public void stop() {
        if (mListener != null) {
            mListener.exit();
        }
    }

    private static class ClientListener extends Thread {
        private ServerSocket server;
        private boolean done = false;

        private ClientListener(int port) throws IOException {
            server = new ServerSocket(port);
            System.out.println("服务器信息：" + server.getInetAddress() + " P:" + server.getLocalPort());
        }

        @Override
        public void run() {
            super.run();

            System.out.println("服务器准备就绪～");
            // 等待客户端连接
            do {
                // 得到客户端
                Socket client;
                try {
                    client = server.accept();
                } catch (IOException e) {
                    continue;
                }
                // 客户端构建异步线程
                ClientHandler clientHandler = new ClientHandler(client);
                // 启动线程
                clientHandler.start();
            } while (!done);

            System.out.println("服务器已关闭！");
        }

        void exit() {
            done = true;
            try {
                server.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 客户端消息处理
     */
    private static class ClientHandler extends Thread {
        private Socket socket;
        private boolean flag = true;

        ClientHandler(Socket socket) {
            this.socket = socket;
        }

        @Override
        public void run() {
            super.run();
            System.out.println("新客户端连接：" + socket.getInetAddress() +
                    " P:" + socket.getPort());

            try {
                // 得到打印流，用于数据输出；服务器回送数据使用
                PrintStream socketOutput = new PrintStream(socket.getOutputStream());
                // 得到输入流，用于接收数据
                BufferedReader socketInput = new BufferedReader(new InputStreamReader(
                        socket.getInputStream()));

                do {
                    // 客户端拿到一条数据
                    String str = socketInput.readLine();
                    if ("bye".equalsIgnoreCase(str)) {
                        flag = false;
                        // 回送
                        socketOutput.println("bye");
                    } else {
                        // 打印到屏幕。并回送数据长度
                        System.out.println(str);
                        socketOutput.println("回送：" + str.length());
                    }

                } while (flag);

                socketInput.close();
                socketOutput.close();

            } catch (Exception e) {
                System.out.println("连接异常断开");
            } finally {
                // 连接关闭
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

            System.out.println("客户端已退出：" + socket.getInetAddress() +
                    " P:" + socket.getPort());
        }
    }
}
```



## 相关工具类

### TCPConstants

```java
public class TCPConstants {
    // 服务器固化UDP接收端口
    public static int PORT_SERVER = 30401;
}
```

### UDPConstants

```java
public class UDPConstants {
    // 服务器固化UDP接收端口
    public static int PORT_SERVER = 30201;
    // 客户端回送端口
    public static int PORT_CLIENT_RESPONSE = 30202;

    // UDP 密码口令
    public static String UDP_PASSWORD_KEY = "maizi";
}

```

### JsonSerilizable

```java
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.TypeReference;

public class JsonSerilizable {
    /* 将HashMap序列化为字符串存入json文件中 */
    public static String serilizableForMap(Object objMap) throws IOException {

        String listString = JSON.toJSONString(objMap, true);// (maps,CityEntity.class);
        return listString;
    }

    /* 将json文件中的内容读取出来，反序列化为HashMap */
    public static <T, K> HashMap<K, T> deserilizableForMapFromFile(String listString2, Class<T> clazz)
            throws IOException {

        Map<K, T> map = JSON.parseObject(listString2, new TypeReference<Map<K, T>>() {
        });

        return (HashMap<K, T>) map;
    }

    public static byte[] changeMapToByte(Map<String, Object> map) {

        byte[] bytes = null;
        try {
            bytes = JsonSerilizable.serilizableForMap(map).getBytes();
        } catch (Exception e) {
            System.out.println("map到byte[]转换异常");
            e.printStackTrace();
        }

        return bytes;
    }

    public static Map<String, Object> changeByteToMap(byte[] bytes) {
        Map<String, Object> retmap = null;
        try {
            if (bytes != null) {
                retmap = JsonSerilizable.deserilizableForMapFromFile(new String(bytes), Object.class);
            } else {
                System.out.println("changeByteToMap中bytes为null");
            }
        } catch (Exception e) {
            System.out.println("byte到map转换异常");
            e.printStackTrace();
        }
        return retmap;
    }

}
```

### ServerInfo

```java
public class ServerInfo {
    private String sn;
    private int port;
    private String address;

    public ServerInfo(int port, String ip, String sn) {
        this.port = port;
        this.address = ip;
        this.sn = sn;
    }
   .....
```

### pom.xml

```xml
<dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>fastjson</artifactId>
      <version>1.2.51</version>
 </dependency>
```

# 运行结果

## 服务端

```java
服务器信息：0.0.0.0/0.0.0.0 P:30401
服务器准备就绪～
UDPProvider Started.
60360
UDPProvider response to:192.168.0.7     port:30202      dataLen:79
新客户端连接：/192.168.0.7 P:62086
sssss
```

## 客户端

```java
UDPSearcher Started.
UDPSearcher start listen.
UDPSearcher sendBroadcast started.
UDPSearcher sendBroadcast finished.
UDPSearcher Finished.
UDPSearcher listener finished.
Server:ServerInfo{sn='cdb9cdb4-6772-4e9b-8414-3dea10de9de3', port=30401, address='192.168.0.7'}
已发起服务器连接，并进入后续流程～
客户端信息：/127.0.0.1 P:62085
服务器信息：/192.168.0.7 P:30401
sssss
回送：5
```



# 说明

这边的服务端目前只可以接受数据， 无法发送数据，后期在进行升级处理。 
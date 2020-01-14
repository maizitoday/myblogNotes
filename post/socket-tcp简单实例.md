---
title:       "Socket-TCP简单实例"
subtitle:    ""
description: ""
date:        2020-01-06
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["网络编程"]
categories:  ["Tech" ]
---

[TOC]

转载地址：https://www.bilibili.com/video/av75603750?p=3  《Socket网络编程进阶与实战》系列视频

# 服务端

```java
package networkdemo;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-01-06 20:28:09
 * @LastEditTime : 2020-01-06 22:01:11
 * @LastEditors  : 麦子
 */

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.net.ServerSocket;
import java.net.Socket;

public class TCPService {
    public static void main(String[] args) throws IOException {
        ServerSocket server = new ServerSocket(2000);

        System.out.println("服务器准备就绪");
        System.out.println("服务器信息: " + server.getInetAddress() + " P: " + server.getLocalPort());

        // 等待客户端连接
        for (;;) {
            Socket client = server.accept();
            ClientHandler clientHandler = new ClientHandler(client);
            clientHandler.start();
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
            System.out.println("新客户端连接: " + socket.getInetAddress() + " P:" + socket.getPort());
            try {

                PrintStream socketOutPut = new PrintStream(socket.getOutputStream());

                // 得到输入流，用于接收数据
                BufferedReader socketInput = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                do {

                    // 客户端拿到一条数据
                    String str = socketInput.readLine();
                    if ("bye".equalsIgnoreCase(str)) {
                        flag = false;
                        // 回送
                        socketOutPut.println("bye");
                    } else {
                        // 打印到屏幕，并回送数据长度
                        System.out.println(str);
                        socketOutPut.println("服务器回送: " + str.length());
                    }

                } while (flag);

                socketInput.close();
                socketOutPut.close();

            } catch (Exception e) {
                System.out.println("连接异常断开");
            } finally {
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

            System.out.println("客户端已退出: " + socket.getInetAddress() + " P:" + socket.getPort());

        }
    }
}
```

# 客服端

```java
package networkdemo;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-01-06 20:29:40
 * @LastEditTime : 2020-01-06 22:02:45
 * @LastEditors  : 麦子
 */

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.PrintStream;
import java.net.Inet4Address;
import java.net.InetSocketAddress;
import java.net.Socket;
import java.net.SocketException;
import java.net.UnknownHostException;
import java.util.concurrent.TimeUnit;

public class TCPClient {

    public static void main(String[] args) throws UnknownHostException, IOException {
        Socket socket = new Socket();
        // 超时时间
        socket.setSoTimeout(3_000);
        // 连接本地， 端口2000，超时时间是3000
        socket.connect(new InetSocketAddress(Inet4Address.getLocalHost(), 2_000), 3_000);

        System.out.println("已发起服务器连接，并进入后续流程");
        System.out.println("客户端信息: " + socket.getLocalAddress() + "  P:" + socket.getLocalPort());
        System.out.println("服务器信息: " + socket.getInetAddress() + " P:" + socket.getPort());

        try {
            // 发送接收数据
            todo(socket);
        } catch (Exception e) {
            System.out.println("异常关闭");
        }

        socket.close();
        System.out.println("客户端已退出");

    }

    //
    private static void todo(Socket client) throws IOException, InterruptedException {
        // 构建键盘输入流
        InputStream in = System.in;
        BufferedReader input = new BufferedReader(new InputStreamReader(in));

        // 得到Socket输出流，并转换为打印流
        OutputStream outputStream = client.getOutputStream();
        PrintStream socketPrintStream = new PrintStream(outputStream);

        // 从服务器获取数据
        InputStream inputStream = client.getInputStream();
        BufferedReader socketBufferedReader = new BufferedReader(new InputStreamReader(inputStream));
        boolean flag = true;
        do {
            // 键盘读取一行
            String str = input.readLine();
            // 发送到服务器
            socketPrintStream.println(str);

            System.out.println("我在等待-----------------");
            TimeUnit.SECONDS.sleep(2);

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

# 运行结果



```java
服务器准备就绪
服务器信息: 0.0.0.0/0.0.0.0 P: 2000
新客户端连接: /127.0.0.1 P:63223
我是麦子
```

```
已发起服务器连接，并进入后续流程
客户端信息: /127.0.0.1  P:63223
服务器信息: localhost/127.0.0.1 P:2000
我是麦子
我在等待-----------------
服务器回送: 4
```



# 注意

TCP上面的这种聊天是一种，发送信息， 然后等待服务器给的回复的模式。他是必须等待那边的返回。优化的在后面博客进行处理。 


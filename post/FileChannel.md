---
title:       "FileChannel和MappedByteBuffer"
subtitle:    ""
description: "处理IO场景需求"
date:        2020-03-18
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["网络编程","java基础"]
categories:  ["Tech" ] 
---

[TOC]

**说明文章资料来源：尚硅谷韩顺平Netty视频教程（2019发布） https://www.bilibili.com/video/av76227904?p=36**

# 常见方法

FileChannel 主要用来对本地文件进行 IO 操作，常见的方法有

```java
public int read(ByteBuffer dst) ，从通道读取数据并放到缓冲区中
public int write(ByteBuffer src) ，把缓冲区的数据写到通道中 
public long transferFrom(ReadableByteChannel src, long position, long count)，从目标通道中复制数据到当前通道
public long transferTo(long position, long count, WritableByteChannel target)，把数据从当前通道复制给目标通道
```

# 应用实例 1-本地文件写数据

```java
package com.nio;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2020-03-18 10:30:03
 * @LastEditTime: 2020-03-18 10:38:22
 * @LastEditors: 麦子
 */

import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class FileChannelDemo {

    public static void main(String[] args) throws IOException {
        String str = "hello,尚硅谷"; // 创建一个输出流->
        FileOutputStream fileOutputStream = new FileOutputStream("/Users/maizi/Desktop/1.txt");

        // 通过 fileOutputStream 获取 对应的 FileChannel //这个 fileChannel 真实 类型是
        FileChannel fileChannel = fileOutputStream.getChannel();

        // 创建一个缓冲区 ByteBuffer
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

        // 将 str 放入 byteBuffer
        byteBuffer.put(str.getBytes());

        // 对 byteBuffer 进行 flip
        byteBuffer.flip();

        // 将 byteBuffer 数据写入到 fileChannel
        fileChannel.write(byteBuffer);
        fileOutputStream.close();
    }

}
```

# 应用实例 2-本地文件读数据

```java
// 创建文件的输入流
File file = new File("/Users/maizi/Desktop/1.txt");
FileInputStream fileInputStream = new FileInputStream(file);

// 通过 fileInputStream 获取对应的 FileChannel -> 实际类型
FileChannel fileChannel = fileInputStream.getChannel();

// 创建缓冲区
ByteBuffer byteBuffer = ByteBuffer.allocate((int) file.length());

// 将 通道的数据读入到 Buffer
fileChannel.read(byteBuffer);

// 将 byteBuffer 的 字节数据 转成 String
System.out.println(new String(byteBuffer.array()));
fileInputStream.close();

```

# 应用实例 3-使用一个 Buffer 完成文件读取、写入

使用 FileChannel(通道) 和 方法 read , write，完成文件的拷贝

```java
 public static void main(String[] args) throws IOException {

        FileInputStream fileInputStream = new FileInputStream("/Users/maizi/Desktop/1.txt");
        FileChannel fileChannel01 = fileInputStream.getChannel();

        FileOutputStream fileOutputStream = new FileOutputStream("/Users/maizi/Desktop/2.txt");
        FileChannel fileChannel02 = fileOutputStream.getChannel();

        ByteBuffer byteBuffer = ByteBuffer.allocate(512);

        while (true) { // 循环读取
            //这里有一个重要的操作，一定不要忘了
            /* 
              public final Buffer clear() 
                { 
                     position = 0; 
                     limit = capacity; 
                     mark = -1; 
                     return this;
                }
            */
            byteBuffer.clear(); // 清空 buffer
            int read = fileChannel01.read(byteBuffer);
            System.out.println("read =" + read);
            if (read == -1) { // 表示读完
                break;
            } // 将 buffer 中的数据写入到 fileChannel02 -- 2.txt
            byteBuffer.flip();
            fileChannel02.write(byteBuffer);

        }

        // 关闭相关的流
        fileInputStream.close();
        fileOutputStream.close();

    }
```

# 应用实例 4-拷贝文件 transferFrom 方法

```java
public class FileChannelDemo {
    public static void main(String[] args) throws IOException {
        FileInputStream fileInputStream = new FileInputStream("/Users/maizi/Desktop/1.txt");
        FileOutputStream fileOutputStream = new FileOutputStream("/Users/maizi/Desktop/3.txt");

        // 获取各个流对应的 filechannel
        FileChannel sourceCh = fileInputStream.getChannel();
        FileChannel destCh = fileOutputStream.getChannel();

        // 使用 transferForm 完成拷贝
        destCh.transferFrom(sourceCh, 0, sourceCh.size()); // 关闭相关通道和流
        sourceCh.close();
        destCh.close();
        fileInputStream.close();
        fileOutputStream.close();
    }
} 
```

# MappedByteBuffer

```java
RandomAccessFile randomAccessFile = new RandomAccessFile("/Users/maizi/Desktop/1.txt", "rw"); // 获取对应的通道
        FileChannel channel = randomAccessFile.getChannel();

        /**
         * * 参数 
         * 1: FileChannel.MapMode.READ_WRITE 使用的读写模式 * 参数 
         * 2： 0 ： 可以直接修改的起始位置 * 参数
         * 3:  5: 是映射到内存的大小(不是索引位置) ,
               即将 1.txt 的多少个字节映射到内存 * 可以直接修改的范围就是 0-5 * 实际类型DirectByteBuffer
         * 
         */
        MappedByteBuffer mappedByteBuffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, 5);

        mappedByteBuffer.put(0, (byte) 'H');
        mappedByteBuffer.put(3, (byte) '9');
        //mappedByteBuffer.put(5, (byte) 'Y');// IndexOutOfBoundsException

        randomAccessFile.close();
        System.out.println("修改成功~~");

原文件内容：hello,尚硅谷

程序运行后打印结果： Hel9o,尚硅谷

```


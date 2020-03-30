---
title:       "RandomAccessFile"
subtitle:    ""
description: "RandomAccessFile类使用说明"
date:        2020-03-30
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["RandomAccessFile","java基础"]
categories:  ["Tech" ] 
---

[TOC]

原文链接：https://blog.csdn.net/qq496013218/article/details/69397380

# 简述

Java中的RandomAccessFile提供了对文件的读写功能。RandomAccessFile 虽然属于java.io下的类，但它不是InputStream或者OutputStream的子类；它也不同于FileInputStream和FileOutputStream。 FileInputStream 只能对文件进行读操作，而FileOutputStream 只能对文件进行写操作；但是，RandomAccessFile 与输入流和输出流不同之处就是RandomAccessFile可以访问文件的任意地方同时支持文件的读和写，并且它支持随机访问。RandomAccessFile包含InputStream的三个read方法，也包含OutputStream的三个write方法。同时RandomAccessFile还包含一系列的readXxx和writeXxx方法完成输入输出。
RandomAccessFile父类：java.lang.Object

# 构造函数

```java
创建随机访问文件流，以从File参数指定的文件中读取，并可选择写入文件。
RandomAccessFile(File file, String mode)

创建随机访问文件流，以从中指定名称的文件读取，并可选择写入文件。
RandomAccessFile(String name, String mode)
 
```

# 构造函数中mode参数传值介绍

- [ ] r 代表以只读方式打开指定文件 
- [ ] rw 以读写方式打开指定文件 
- [ ] rws 读写方式打开，并对内容或元数据都同步写入底层存储设备
- [ ] rwd 读写方式打开，对文件内容的更新同步更新至底层存储设备

RandomAccessFile包含了一个对象记录的指针，用于标识当前流的读写位置RandomAccessFile包含两个方法来操作文件记录指针。文件指针可以通过getFilePointer方法读取，并由seek方法设置。

- [ ] **long getFilePoint()**:设置文件指针偏移，从该文件的开头测量，发生下一次读取或写入。(前面是文档原文翻译通俗一点就是：返回文件记录指针的当前位置,不指定指针的位置默认是0。

  

- [ ] **void seek(long pos)**:设置文件指针偏移，从该文件的开头测量，发生下一次读取或写入。（前面是文档原文翻译通俗一点就是：将文件记录指针定位到pos位置。

# 以下是读取和写入文件的实例

## 1、读指定文件的内容，并且输出控制台

```java
    public static void main(String[] args) throws Exception{
        RandomAccessFile raf=new RandomAccessFile("G:\\java-lambda\\work.txt","r");
        byte[] buff = new byte[1024];
        int len = 0;
        while ((len = raf.read(buff,0,1024))!=-1){
            System.out.println(new String(buff,0,len));
        }
     }
```

## 2、通过指定记录指针的位置及跳过的字节数，输出内容

```java
public static void RandomAccess(String path,String mode,long seek,int skipBytes){
        RandomAccessFile raf = null;
        try {
            raf = new RandomAccessFile(path,mode);
            raf.writeBytes("If you interesting to me,please give the kiss to me!");
            raf.seek(seek);//指定记录指针的位置
            //System.out.println(raf.readLine());//使用seek指针指向0，readLine读取所有内容
            raf.getFilePointer();//获取指针的位置
            raf.skipBytes(skipBytes);//跳过的字节数
            byte[] buff = new byte[1024];
            int len = 0;
            while ((len = raf.read(buff))>0){
                System.out.println(new String(buff,0,len));
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            if(raf != null){
                try {
                    raf.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

## 3、在内容后面插入一个字符串，并输出

```java
public static void RandomAccessTem(String path,String mode,long seek,int skipBytes){
        RandomAccessFile raf = null;
        try {
            raf = new RandomAccessFile(path,mode);
            raf.writeBytes("If you interesting to me,please give the kiss to me!");
            raf.seek(raf.length());//通过raf.length()获取总长度，记录指针指向最后
            raf.write("oh yes！".getBytes());//在最后插入oh yes!s
            raf.seek(0);//记录指针指向初始位置
            byte[] buff = new byte[1024];
            int len = 0;
            while ((len = raf.read(buff))>0){
                System.out.println(new String(buff,0,len));
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            if(raf != null){
                try {
                    raf.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

## 4、通过临时文件插入内容

```java
public static void RandomAccessTemp(String path,String mode){
        File tempFile = null;
        RandomAccessFile raf = null;
        FileOutputStream out = null;
        FileInputStream input = null;
        try {
            //创建临时文件
            tempFile =File.createTempFile("temp","_temp");
            //在JVM进程退出的时候删除文件,通常用在临时文件的删除
            tempFile.deleteOnExit();
            //创建一个临时文件夹来保存插入点后的数据
            out = new FileOutputStream(tempFile);
            input=new FileInputStream(tempFile);
            //指定文件路径和读写方式
            raf = new RandomAccessFile(path,mode);
            //raf.writeBytes("If you interesting to me,please give the kiss to me!");
            //记录指针指向初始位置
            raf.seek(0);
            byte[] buff = new byte[1024];
            int len = 0;
            //循环读取插入点后的内容
            //读取文件，存入字节数组buff，返回读取到的字符数赋值给len,默认每次将buff数组装满
            while ((len = raf.read(buff))>0){
                // 将读取的数据写入临时文件中
                out.write(buff,0,len);
                System.out.println(new String(buff,0,len));
            }
            //记录指针指向初始位置
            raf.seek(0);
            //插入指定内容
            raf.writeBytes("my name's Toms!");
            //在临时文件中插入指定内容
            while ((len = input.read(buff))>0){
                raf.write(buff,0,len);
            }
            out.close();
            input.close();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            if(raf != null){
                try {
                    raf.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

## 方法摘要

```java
void close() ：关闭此随机访问文件流并释放与该流关联的所有系统资源。

FileChannel getChannel() ： 返回与此文件关联的唯一 FileChannel 对象。

FileDescriptor getFD() ：返回与此流关联的不透明文件描述符对象。

long getFilePointer() ：返回此文件中的当前偏移量。

long length() ：返回此文件的长度。

int read() ：从此文件中读取一个数据字节。

int read(byte[] b) ： 将最多 b.length 个数据字节从此文件读入 byte 数组。

int read(byte[] b, int off, int len) ：将最多 len 个数据字节从此文件读入 byte 数组。

boolean readBoolean() ：从此文件读取一个 boolean。

byte readByte() ：从此文件读取一个有符号的八位值。

char readChar() ：从此文件读取一个字符。

double readDouble() ：从此文件读取一个 double。

float readFloat() ：从此文件读取一个 float。

void readFully(byte[] b) ： 将 b.length 个字节从此文件读入 byte 数组，并从当前文件指针开始。

void readFully(byte[] b, int off, int len) ：将正好 len 个字节从此文件读入 byte 数组，并从当前文件指针开始。

int readInt() ：从此文件读取一个有符号的 32 位整数。

String readLine() ：从此文件读取文本的下一行。

long readLong() ： 从此文件读取一个有符号的 64 位整数。

short readShort() ：从此文件读取一个有符号的 16 位数。

int readUnsignedByte() ：从此文件读取一个无符号的八位数。

int readUnsignedShort() ：从此文件读取一个无符号的 16 位数。

String readUTF() ：从此文件读取一个字符串。

void seek(long pos) ：设置到此文件开头测量到的文件指针偏移量，在该位置发生下一个读取或写入操作。

void setLength(long newLength) ：设置此文件的长度。

int skipBytes(int n) ：尝试跳过输入的 n 个字节以丢弃跳过的字节。

void write(byte[] b) ： 将 b.length 个字节从指定 byte 数组写入到此文件，并从当前文件指针开始。

void write(byte[] b, int off, int len) ：将 len 个字节从指定 byte 数组写入到此文件，并从偏移量 off 处开始。

void write(int b) ：向此文件写入指定的字节。

void writeBoolean(boolean v) ：按单字节值将 boolean 写入该文件。

void writeByte(int v) ：按单字节值将 byte 写入该文件。

void writeBytes(String s) ： 按字节序列将该字符串写入该文件。

void writeChar(int v) ：按双字节值将 char 写入该文件，先写高字节。

void writeChars(String s) ： 按字符序列将一个字符串写入该文件。

void writeDouble(double v) ：使用 Double 类中的 doubleToLongBits 方法将双精度参数转换为一个 long，然后按八字节数量将该 long 值写入该文件，先定高字节。

void writeFloat(float v) ：使用 Float 类中的 floatToIntBits 方法将浮点参数转换为一个 int，然后按四字节数量将该 int 值写入该文件，先写高字节。

void writeInt(int v) ：按四个字节将 int 写入该文件，先写高字节。

void writeLong(long v) ：按八个字节将 long 写入该文件，先写高字节。

void writeShort(int v) ：按两个字节将 short 写入该文件，先写高字节。

void writeUTF(String str) ：使用 modified UTF-8 编码以与机器无关的方式将一个字符串写入该文件。
 
```

# 应用场景

1、 是JAVA I/O流体系中功能最丰富的文件内容访问类，它提供了众多方法来访问文件内容。

2、 由于可以自由访问文件的任意位置，所以如果需要访问文件的部分内容，RandomAccessFile将是更好的选择。

3、 可以用来访问保存数据记录的文件，文件的记录的大小不必相同，但是其大小和位置必须是可知的。

4、 断点续传，使用seek()方法不断的更新下载资源的位置。

5、 向10G文件末尾插入指定内容，或者向指定指针位置进行插入或者修改内容。
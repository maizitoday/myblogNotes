---
title:       "IO类框架"
subtitle:    ""
description: ""
date:        2019-12-31
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["java基础"]
categories:  ["Tech" ]
---

[TOC]

转载地址：https://blog.csdn.net/jssg_tzw/article/details/77991749

转载地址：https://blog.csdn.net/a18716374124/article/details/76504706

转载地址： https://www.cnblogs.com/dengyungao/p/7525098.html

# 概念

流（Stream）的概念来源于UNIX中的管道（pipe）概念，在unix中，管道是一条不间断的字节流，用来实现程序和进程间的通信，或者读写外围设备、外部文件等。流，必须有源端和目的端，可以是文件，内存或者网络等。流的创建是为了更方便的处理数据的输入输出。简单的来说：输入流就是从外部文件输入到内存，输出流主要是从内存输出到文件。我们用Eclipse开发小程序在控制台输入数据就属于输入流，即从控制台输入到内存。

# 结构图

![21134209_eZJr](/img/21134209_eZJr.jpg)

# 字节流和字符流的区别

字节流处理单元为1个字节，操作字节和字节数组，而字符流处理的单元为2个字节的Unicode字符，分别操作字符、字符数组或字符串。程序中的输入输出都是以流的形式保存的，流中保存的实际上全都是字节文件。所有文件的储存是都是字节（byte）的储存，在磁盘上保留的并不是文件的字符而是先把字符编码成字节，再储存这些字节到磁盘。在读取文件（特别是文本文件）时，也是一个字节一个字节地读取以形成字节序列。**那么既然磁盘存储都是按字节，内存中处理为何还需要字符流呢？**字符流是由Java虚拟机将字节转化为2个字节的Unicode字符为单位的字符而成的，所以它对多国语言支持性比较好！如果是音频文件、图片、歌曲，就用字节流好点，如果是关系到中文（文本）的，用字符流好点！

**那开发中究竟用字节流好还是用字符流好呢**？

在所有的硬盘上保存文件或进行传输的时候都是以字节的方法进行的，包括图片也是按字节完成，而字符是只有在内存中才会形成的，所以使用字节的操作是最多的。我们建议尽量尝试使用字符流，一旦程序无法成功编译，就不得不使用面向字节的类库，即字节流。

# 分类

## 按流方向分类

从流的方向上可分为两类(在java中是站在程序角度来区分流的方向，将数据读取到程序中就是输入流；反之，将程序中的数据写出去就是输出流)：
- 输入流： 从数据源中将数据读取到程序中的流。
- 输出流：程序将数据写入到目的地的流。

## 按流的数据类型分类

- 字节流： 以8位的字节形式来读写的流。他们的标志是名称以Stream结尾。InputStream与OutputStream分别是所有字节输入流与字节输出流的抽象父类。
- 字符流： 以字符形式来读写的流。它们的标志是名称以Reader或者Writer结尾。并且Reader和Writer分别是所有字符输入流与字符输出流的抽象父类。



# 流式部分

## 字节流

字节流主要是操作byte类型数据，以byte数组为准，主要操作类就是OutputStream、InputStream。两者都是抽象类，是字节输入、输出流的父类。以InputStream为例，最常用到的子类有：FileInputStream，ByteArrayInputStream，ObjectInputStream，BufferedInputStream、DataInputStream、PushbackInputStream。

### 缓冲字节流好处

BufferedInputStream和InputStream类都是字节流，而更确切地说，BufferedInputStream是缓冲字节流。

InputStream是不带缓冲的操作，每读一个字节就要写入一个字节，由于涉及磁盘的IO操作相比内存的操作要慢很多，所以不带缓冲的流效率很低。BufferedInputStream带缓冲的流，可以一次读很多字节，但不向磁盘中写入，只是先放到内存里。等凑够了缓冲区大小的时候一次性写入磁盘，这种方式可以减少磁盘操作次数，速度就会提高很多！这是两者的区别。

### 字节输入流

#### 结构图

![21134209_DSCU](/img/21134209_DSCU.gif)



#### 概述

InputStream: 字节输入流、是所有字节输入流的父类、为所有字节输入流提供一个标准、和基本的与读取字节有关的方法及简单的实现

**1.   FileInputStream**

文件字节输入流、以字节的形式将文件中数据读取到程序中、进行下一步操作;

**2.  FilterInputStream**

过滤器字节输入流、仅仅是重写InputStream方法、为字节输入处理流提供标准

| 相关子类            | 说明                                                         |
| :------------------ | :----------------------------------------------------------- |
| BufferedInputStream | 缓冲字节输入流、装饰类、用于装饰节点类型的字节输入流、为其提供缓冲功能、减少访问次数、提高效率; |
| DataInputStream     | 数据字节输入流、装饰类、装饰字节节点输入流、一般与DataOutputStream结合使用、用于读取DataOutputStream写入到目的地中的java基础类型的数据; |
| PushbackInputStream | 在JAVA IO中所有的数据都是采用顺序的读取方式，即对于一个输入流来讲都是采用从头到尾的顺序读取的，如果     在输入流中某个不需要的内容被读取进来，则只能通过程序将这些不需要的内容处理掉，为了解决这样的处理问题，在JAVA中提供了一种回退输入流PushbackInputStream可以把读取进来的某些数据重新回退到输入流的缓冲区之中; |

**3.  ObjectInputStream**

对象字节输入流、与ObjectOutputStream结合使用、用于将使用ObjectOutputStream写入目的地的java对象（一般是javabean）读取到程序中、进行下一步操作;

**4.  PipedInputStream**

管道字节输入流、必须与PipedOutputStream（管道字节输出流）结合使用、用于线程之间的通信;

**5.  ByteArrayInputStream**

字节数组输入流、用于读取其内置缓存字节数组中的字节。内存缓存字节数组中数据的来源是在构建ByteArrayInputStream时传入的字节数组;

**6.  StringBufferedInputStream** 

已过时;

**7.  SequenceInputStream**

表示其他输入流的逻辑串联。它从输入流的有序集合开始，并从第一个输入流开始读取，直到到达文件末尾，接着从第二个输入流读取，依次类推，直到到达包含的最后一个输入流的文件末尾为止





### 字节输出流

#### 结构图

![21134209_WFrb](/img/21134209_WFrb.gif)

#### 概述

OutputStream: 字节输出流、本身是一个抽象类、与InputStream作用一样、为所有字节输出流提供一个标准、定义了一些基本写入字节的方法与简单实现:

**1.  FileOutputStream**

文件字节输出流、将字节写入指定的文件中;

**2.  FilterOutputStream**

过滤字节输出流、仅仅重写了父类OutputStream的方法、是字节输出处理流的父类

| 相关子类             | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| BufferedOutputStream | 缓冲字节输出流、装饰类、为低级字节输出流添加缓冲功能、提高效率、减少访问次数; |
| DataOutputStream     | 数据字节输出流、装饰类。用于装饰节点输出流、功能是将java基础类型以一种与系统无关的方式写入到目的地中、可用DataInputStream读取; |
| PrintStream          | 字节打印流、装饰类、这是一个功能很强的流、对低级字节输出流进行装饰、提供打印各种数据类型的功能、打印的目的地不仅仅是控制台、可以是文件、内存、网络等、同时还可以指定自动flush功能; |

**3.  ObjectOutputStream**

对象字节输出流、用于将java的对象（一般是javabean）写入输出流管道中、存储到某介质中、可使用ObjectInputStream读取被其写入的对象;

**4.  PipedoutputStream**

管道字节输出流、在上边介绍过了、必须与PipedInputStream结合使用、用于线程之间的通信;

**5.  ByteArrayOutputStream**

字节数组输出流、用于将字节写入到其本身所带的一个内置缓存字节数组中、可将其内置缓存字节数组中的字节转化成字符串到程序中、也可以将它写入到另一个字节输出流中;





## 字符流

在程序中一个字符等于两个字节，java提供了Reader、Writer两个专门操作字符流的类。两者同样是抽象类。已Reader为例，其常用的子类有：[BufferedReader](http://blog.csdn.net/sinat_26888717/article/details/47999637),[CharArrayReader](http://blog.csdn.net/sinat_26888717/article/details/47999637),[FilterReader](http://blog.csdn.net/sinat_26888717/article/details/47999637),[InputStreamReader](http://blog.csdn.net/sinat_26888717/article/details/47999637),[PipedReader](http://blog.csdn.net/sinat_26888717/article/details/47999637),[StringReader](http://blog.csdn.net/sinat_26888717/article/details/47999637) 。

其中FilterReader是抽象类，其实例化的类为PushbackReader。

文章开头图中的ByteArrayReader其实应为此处的CharArrayReader，其构造函数的参数要接收字符数组。

PipedReader是管道输入流，需要和PipedWriter合起来用。

StringReader是源为一个字符串的字符流，即其构造函数的参数要接收字符串。

**InputStreamReader 是字节流通向字符流的桥梁**：它使用指定的 [charset](http://blog.csdn.net/sinat_26888717/article/details/47999637)读取字节并将其解码为字符。注意：FileReader是InputStreamReader的子类！InputStreamReader构造方法的参数是什么呢？是InputStream！**也就是说可以将字节输入流封装到InputStreamReader中，实现字节流到字符流的转换！** 另外，InputStreamReader和OutputStreamWriter这两个类是字节流和字符流之间的适配器类，它们承担编码转换的任务。

BufferedReader从字符输入流中读取文本，缓冲各个字符，从而实现字符、数组和行的高效读取。 其构造方法的参数为Reader子类。



### 字符输入流

#### 结构图

![21134209_bOjq](/img/21134209_bOjq.gif)



#### 概述

Reader、意义与InputStream相同、为所有字符输入流提供一个标准、只有基本的读取方法的定义和简单的实现

**1.  BufferedReader**

缓冲字符输入流。装饰类、为低级字符输入流提供缓冲功能、提高效率。并且这里不再继承FilterReader、而是直接继承Reader;

**2.  InputStreamReader**

字节转换流、将字节流转换成字符流、不仅仅可以将低级字节流输入流转换成字符输入流、还可以将高级字节输入流转换成字符输入流、可以指定字节转成字符时使用的编码、所有具有指定编码或者编码集的字符输入流内部都是调用了此流来转换底层流;

| 相关子类   | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| FileReader | 文件字符输入流、以字符的形式读取文件中的内容。当文件内容是字节时则可能会有问题; |

**3.  PipedReader**

管道字符输入流：必须与PipedWriter结合使用、用于线程之间的通信;

**4.  CharArrayReader**

字符数组输入流、将其内置字符缓存数组中的数据（通过构造方法传入）读取到程序中。内置字符缓存数组中数据的来源是在构造CharArrayReader时传入的字符数组;

**5. FilterReader**

字符过滤输入流、本事是一个抽象类、为所有装饰类提供一个标准、只是简单重写了父类Reader的所有方法、要求子类必须重写核心方法、和提供具有自己特色的方法、这里没有像字节流那样有很多的子类来实现不同的功能、可能是因为字符流本来就是字节流的一种装饰、所以在这里没有必要再对其进行装饰、只是提供一个扩展的接口。

| 相关子类       |                                                              |
| -------------- | ------------------------------------------------------------ |
| PushbackReader | java.io.PushbackReader与前面提到的PushbackInputStream类似，都拥有一个PushBack缓冲区，只不过PushbackReader所处理的是字符。从这个对象读出数据后，如果愿意的话，只要PushBack缓冲区没有满，就可以使用unread()将数据推回流的前端。 |

**6.  StringReader**

StringReader 是读, 从一个String中读取,所以需要一个String ,通过构造方法传递。





### 字符输出流

#### 结构图

![21134209_ugym](/img/21134209_ugym.gif)

#### 概述

Writer ：字符输出流、是所有字符输出流的父类。与OutputStream意义一样、提供一个标准、一些基本写入的方法和简单的实现;

**1.  BufferedWriter**

缓存字符输出流、用于装饰低级字符输出流、为其提供缓冲功能、提高效率;

**2.  OutputStreamWriter**

字节输出流转换成字符输出流、用于将字节输出流转化成字符输出流。可以使用指定编码转化。为提高效率、通常使用BufferedWriter对其包装;

| 相关子类   | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| FileWriter | 文件字符输出流、继承OutputStreamWriter、用于将字符写入到指定文件中; |

**3.  PrintWriter**

字符打印流、同样很强大、可以将各种数据以字符的形式打印到指定目的地中、本身也是个装饰流。但是flush与PrintStream有区别;

**4.  PipedWriter**

管道字符输出流、必须与PipedReader结合使用、用于线程之间的通信;

**5.  CharArrayWriter**

字符输出流、用于将字符写入其内置缓存字符数组中。可以将写入的字符缓存数组转换成字符串或者写入其他字符输出流中;

**6.  StringWriter**

在字符串缓冲区中收集输出的字符流，可用于构造字符串， 关闭流无效，关闭后调用其他方法不会报异常

## 常用方法

### 字节流

| **read**()                           | 参见 `InputStream` 的 `read` 方法的常规协定。                |
| ------------------------------------ | ------------------------------------------------------------ |
| **read**(byte[] b, int off, int len) | 从此字节输入流中给定偏移量处开始将各字节读取到指定的 byte 数组中。 |
|                                      |                                                              |
|                                      |                                                              |

### 字符流

| **read**()                              | 读取单个字符。             |
| --------------------------------------- | -------------------------- |
| **read**(char[] cbuf, int off, int len) | 将字符读入数组的某一部分。 |
| **readLine**()                          | 读取一个文本行。           |
|                                         |                            |



## 示例代码



### 不显示close()关闭流

```java
   try (FileInputStream in = new FileInputStream("word.txt");) {// 读取文件
            in.read();// 读取一个字节
        } catch (IOException e1) {
            e1.printStackTrace();
        } // try..catch语句结束后自动关闭in
```



### 注解关掉流

@Cleanup: 该注解能帮助我们自动调用close()方法，很大的简化了代码。

```java
public class LombokTest {

    public static void main(String[] args) throws IOException {
        @Cleanup
        InputStream in = new FileInputStream(args[0]);
        @Cleanup
        OutputStream out = new FileOutputStream(args[1]);
        byte[] b = new byte[10000];
        while (true) {
            int r = in.read(b);
            if (r == -1)
                break;
            out.write(b, 0, r);
        }
    }

}
```

相当于执行完成后， 默认会去调用close()方法。 



# 非流试部分

## File

用于文件或者目录的描述信息，例如生成新目录，修改文件名，删除文件，判断文件所在路径等。

### 构造器

File类共提供了三个不同的构造函数，以不同的参数形式灵活地接收文件和目录名信息。构造函数：

1）File (String   pathname)  

```java
 例:File  f1=new File("FileTest1.txt"); //创建文件对象f1，f1所指的文件是在当前目录下创建的FileTest1.txt
```

2）File (String  parent  ,  String child)

```java
例:File f2=new  File(“D:\\dir1","FileTest2.txt") ;//  注意：D:\\dir1目录事先必须存在，否则异常
```

3）File (File    parent  , String child)

```java
例:File  f4=new File("\\dir3");
          File  f5=new File(f4,"FileTest5.txt");  //在如果 \\dir3目录不存在使用f4.mkdir()先创建
```

### 常用方法

一个对应于某磁盘文件或目录的File对象一经创建， 就可以通过调用它的方法来获得文件或目录的属性。

```java
1）public boolean exists( ) 判断文件或目录是否存在
2）public boolean isFile( ) 判断是文件还是目录 
3）public boolean isDirectory( ) 判断是文件还是目录
4）public String getName( ) 返回文件名或目录名
5）public String getPath( ) 返回文件或目录的路径。
6）public long length( ) 获取文件的长度 
7）public String[ ] list ( ) 将目录中所有文件名保存在字符串数组中返回。
```

File类中还定义了一些对文件或目录进行管理、操作的方法，常用的方法有：

```
1） public boolean renameTo( File newFile );    重命名文件
2） public void delete( );   删除文件
3） public boolean mkdir( ); 创建目录
```

**说明：File类的方法:**

```java
(1) exists()测试磁盘中指定的文件或目录是否存在
(2) mkdir()创建文件对象指定的目录（单层目录）
(3) createNewFile()创建文件对象指定的文件
(4) list()返回目录中所有文件名字符串
```



## RandomAccessFile（随机文件操作）

它的功能丰富，可以从文件的任意位置进行存取（输入输出）操作。

1. RandomAccessFile类的主要功能是完成随机读取功能，可以读取指定位置的内容
2. RandomAccessFile写数据的时候可以将一个字符串写入，读的时候需要一个个的以字节的形式读取出来
3. 如果要想操作文件内容的话，可以使用 RandomAccessFile完成

### 小结

1. RandomAccessFile类的作用
2. RandomAccessFile类操作起来比较麻烦，所以在 IO中会提供专门的输入、输出操作

# 按I/O类型来总体分类

## 1. Memory

1）从/向内存数组读写数据: CharArrayReader、 CharArrayWriter、ByteArrayInputStream、ByteArrayOutputStream

2）从/向内存字符串读写数据 StringReader、StringWriter、StringBufferInputStream

## 2. Pipe管道实现管道的输入和输出（进程间通信）

PipedReader、PipedWriter、PipedInputStream、PipedOutputStream

## 3. File 文件流对文件进行读、写操作

 FileReader、FileWriter、FileInputStream、FileOutputStream

## 4. ObjectSerialization(对象输入、输出)

ObjectInputStream、ObjectOutputStream

## 5. DataConversion数据流 按基本数据类型读、写（处理的数据是Java的基本类型（如布尔型，字节，整数和浮点数））

DataInputStream、DataOutputStream

## 6. Printing 包含方便的打印方法

PrintWriter、PrintStream

## 7. Buffering缓冲  在读入或写出时，对数据进行缓存，以减少I/O的次数

BufferedReader、BufferedWriter、BufferedInputStream、BufferedOutputStream

## 8. Filtering 滤流，在数据进行读或写时进行过滤

FilterReader、FilterWriter、FilterInputStream、FilterOutputStream

## 9. Concatenation合并输入 把多个输入流连接成一个输入流

SequenceInputStream

## 10. Counting计数  在读入数据时对行记数

LineNumberReader、LineNumberInputStream

## 11. Peeking Ahead 通过缓存机制，进行预读

PushbackReader、PushbackInputStream

## 12. Converting between Bytes and Characters 按照一定的编码/解码标准将字节流转换为字符流，或进行反向转换（Stream到Reader,Writer的转换类）

InputStreamReader、OutputStreamWriter

# 按数据来源（去向）分类

## 1、File（文件） 

**FileInputStream, FileOutputStream, FileReader, FileWriter **

## 2、byte[]

**ByteArrayInputStream, ByteArrayOutputStream **

## 3、Char[]:
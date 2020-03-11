---
title:       "ThreadLocal理解"
subtitle:    ""
description: ""
date:        2019-07-09
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["java基础", "并发编程"]
categories:  ["Tech" ]
---



[TOC]

**转载地址：https://www.imooc.com/article/25536**

**转载地址：http://ifeve.com/java-threadlocal的使用/**

# ThreadLocal 是什么

结合我的总结可以这样理解：ThreadLocal提供了线程的局部变量，每个线程都可以通过`set()`和`get()`来对这个局部变量进行操作，但不会和其他线程的局部变量进行冲突，**实现了线程的数据隔离**～。

简要言之：往ThreadLocal中填充的变量属于**当前**线程，该变量对其他线程而言是隔离的。可以把他理解为一个容器。

# Java ThreadLocal的使用

Java中的ThreadLocal类允许我们创建只能被同一个线程读写的变量。因此，如果一段代码含有一个ThreadLocal变量的引用，即使两个线程同时执行这段代码，它们也无法访问到对方的ThreadLocal变量。

# 如何创建ThreadLocal变量

以下代码展示了如何创建一个ThreadLocal变量：

```java
private static ThreadLocal<String> myThreadLocal = new ThreadLocal<String>();
```

我们可以看到，通过这段代码实例化了一个ThreadLocal对象。我们只需要实例化对象一次，并且也不需要知道它是被哪个线程实例化。虽然所有的线程都能访问到这个ThreadLocal实例，但是每个线程却只能访问到自己通过调用ThreadLocal的set()方法设置的值。即使是两个不同的线程在同一个ThreadLocal对象上设置了不同的值，他们仍然无法访问到对方的值。

# 如何访问ThreadLocal变量

一旦创建了一个ThreadLocal变量，你可以通过如下代码设置某个需要保存的值：

```java
myThreadLocal.set("A thread local value”);
```

可以通过下面方法读取保存在ThreadLocal变量中的值：

```java
String threadLocalValue = myThreadLocal.get();
```



# 代码证实

```java
 public interface Sequence {
    
    int getNumber();

    void setNumber(int index);
}
```

```java
public class ClientThread extends Thread {

    private Sequence sequence;

    public ClientThread(Sequence sequence) {
        this.sequence = sequence;
    }

    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            sequence.setNumber(i);
            System.out.println(Thread.currentThread().getName() + " =>" + sequence.getNumber());
        }
    }

}
```

```java
public class SequenceA implements Sequence {

    private static int number = 0;

    private static ThreadLocal<Integer> numberThreadLocal = new ThreadLocal<>();

    @Override
    public int getNumber() {

        // return number;         1号 
        return numberThreadLocal.get();
    }

    @Override
    public void setNumber(int index) {
        // number = number + 1;         1号 
        numberThreadLocal.set(index+1);
    }

    public static void main(String[] args) {
        Sequence sequence = new SequenceA();

        ClientThread thread1 = new ClientThread(sequence);
        ClientThread thread2 = new ClientThread(sequence);
        ClientThread thread3 = new ClientThread(sequence);

        thread1.start();
        thread2.start();
        thread3.start();

    }
}
```

运行效果如下：

```java
Thread-0 =>1
Thread-2 =>1
Thread-1 =>1
Thread-2 =>2
Thread-2 =>3
Thread-0 =>2
Thread-0 =>3
Thread-1 =>2
Thread-1 =>3
```

如果打开注释的代码  1号 ， 运行效果如下：

```java
Thread-1 =>2
Thread-2 =>3
Thread-0 =>2
Thread-2 =>5
Thread-1 =>4
Thread-2 =>7
Thread-0 =>6
Thread-0 =>9
Thread-1 =>8
```

这里就可以看到， number这个变量已经被三个线程在使用了， 使用ThreadLocal可以实现线程隔离操作。

# 典型用途

## 管理数据库的Connection

当时在学JDBC的时候，为了方便操作写了一个简单数据库连接池，需要数据库连接池的理由也很简单，频繁创建和关闭Connection是一件非常耗费资源的操作，因此需要创建数据库连接池～

那么，数据库连接池的连接怎么管理呢？？我们交由ThreadLocal来进行管理。为什么交给它来管理呢？？ThreadLocal能够实现**当前线程的操作都是用同一个Connection，保证了事务！**

```java
public class DBUtil {
    //数据库连接池
    private static BasicDataSource source;

    //为不同的线程管理连接
    private static ThreadLocal<Connection> local;

    static {
        try {
            //加载配置文件
            Properties properties = new Properties();

            //获取读取流
            InputStream stream = DBUtil.class.getClassLoader().getResourceAsStream("连接池/config.properties");

            //从配置文件中读取数据
            properties.load(stream);

            //关闭流
            stream.close();

            //初始化连接池
            source = new BasicDataSource();

            //设置驱动
            source.setDriverClassName(properties.getProperty("driver"));

            //设置url
            source.setUrl(properties.getProperty("url"));

            //设置用户名
            source.setUsername(properties.getProperty("user"));

            //设置密码
            source.setPassword(properties.getProperty("pwd"));

            //设置初始连接数量
            source.setInitialSize(Integer.parseInt(properties.getProperty("initsize")));

            //设置最大的连接数量
            source.setMaxActive(Integer.parseInt(properties.getProperty("maxactive")));

            //设置最长的等待时间
            source.setMaxWait(Integer.parseInt(properties.getProperty("maxwait")));

            //设置最小空闲数
            source.setMinIdle(Integer.parseInt(properties.getProperty("minidle")));

            //初始化线程本地
            local = new ThreadLocal<>();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static Connection getConnection() throws SQLException {
        //获取Connection对象
        Connection connection = source.getConnection();

        //把Connection放进ThreadLocal里面
        local.set(connection);

        //返回Connection对象
        return connection;
    }

    //关闭数据库连接
    public static void closeConnection() {
        //从线程中拿到Connection对象
        Connection connection = local.get();

        try {
            if (connection != null) {
                //恢复连接为自动提交
                connection.setAutoCommit(true);

                //这里不是真的把连接关了,只是将该连接归还给连接池
                connection.close();

                //既然连接已经归还给连接池了,ThreadLocal保存的Connction对象也已经没用了
                local.remove();

            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

}

```

## Spring 是如何解决并发访问的线程安全性问题的

**转载地址：https://www.cnblogs.com/tiancai/p/9627109.html**

springmvc的controller是singleton的（非线程安全的），所在在使用spring开发web 时要注意，默认Controller、Dao、Service都是单例的。由于只有一个Controller的instance，当多个线程调用它的时候，它里面的instance变量就不是线程安全的了，会发生窜数据的问题。当然大多数情况下，我们根本不需要考虑线程安全的问题，比如dao,service等，除非在bean中声明了实例变量。因此，我们在使用spring mvc 的contrller时，应避免在controller中定义实例变量。 

有几种解决方法：
**1、在Controller中使用ThreadLocal变量**
2、在spring配置文件Controller中声明 scope="prototype"，每次都创建新的controller。

# ThreadLocal原理

1. 每个Thread维护着一个ThreadLocalMap的引用

2. ThreadLocalMap是ThreadLocal的内部类，用Entry来进行存储

3. 调用ThreadLocal的set()方法时，实际上就是往ThreadLocalMap设置值，key是ThreadLocal对象，值是传递进来的对象

4. 调用ThreadLocal的get()方法时，实际上就是往ThreadLocalMap获取值，key是ThreadLocal对象

5.  ThreadLocal本身并不存储值**，它只是**作为一个key来让线程从ThreadLocalMap获取value。

正因为这个原理，所以ThreadLocal能够实现“数据隔离”，获取当前线程的局部变量值，不受其他线程影响～

 具体原理查看：https://www.imooc.com/article/25536

 
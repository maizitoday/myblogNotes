---
title:       "Zookeeper-分布式锁"
subtitle:    ""
description: ""
date:        2019-05-13
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["Zookeeper", "分布式锁"]
categories:  ["Tech" ]
---

[TOC]



**说明： 代码实现基于https://www.bilibili.com/video/av39142828?from=search&seid=11914183011254036747视频**

# 分布式锁解决方案

![5-13](/img/5-13.png)



# 实现代码

重现Lock接口，这一块可以提出来作为一个公共工具类。

## Lock接口重写类

```java
package com.example.lockdemo;
/*
 * @Description: 处理Zookeeper分布式锁
 * @Author: 麦子
 * @Date: 2019-05-13 18:33:10
 * @LastEditTime: 2019-05-13 22:18:55
 * @LastEditors: 麦子
 */

import java.util.Collections;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;

import org.I0Itec.zkclient.IZkDataListener;
import org.I0Itec.zkclient.ZkClient;
import org.I0Itec.zkclient.serialize.SerializableSerializer;

/***
 *   思路： 需要锁的话，就需要一个依据来处理。 
 *    
 *   1.  每一个线程来处理业务，都在Zookeeper上面注册一个有序路径节点。 
 *   2.  查看特定目录下面的如我这里就是 ticketLock 这个节点下面的所有生成的子节点，查看这个最小的有序路径节点。
 *   3.  如果是获取到最小的有序节点，就获取到了锁，如果当前线程获取的不是最小的有序节点，那么就暂停当前线程。
 *   4.  当其中另外一条线程处理完成后，就会触发，删除节点的回调， 这个时候在把标记了暂停的那一条线程给释放走下去，在此去获取锁的方法。
 *   5.  从上面的四个步骤可以看出，其中Zookeeper这个节点就是用来控制线程的暂停和释放的关键。每一条线程就相当于一个有序节点，这样就可以看到线程变相的
 *       成为了一个有序的队列形式了。这样的话，执行起来就是按相应的步骤了，一步一步的执行。
 */
public class ZkLockDemo implements Lock {

    private String zkServers = "127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183";
    private ZkClient zkClient = new ZkClient(zkServers, 2000, 1000, new SerializableSerializer());
    private CountDownLatch cd;

    private String beforePath; // 当前请求的节点前一个节点
    private String currentPath; // 当前请求的节点

    private static String LOCK_PATH = "ticketLock";

    /***
     * 判断是否有 ticketLock 目录， 没有就创建
     */
    public ZkLockDemo() {
        if (!this.zkClient.exists("/"+ LOCK_PATH)) {
            this.zkClient.createPersistent(LOCK_PATH);
        }
    }

    @Override
    public void lock() {
        if (tryLock()) {
            return;
        }
        waitForLock();
        // 递归获取锁
        lock();
    }

    /**
     * 让当前线程休眠一段时间
     */
    private void waitForLock() {
        IZkDataListener listener = new IZkDataListener() {

            @Override
            public void handleDataDeleted(String dataPath) throws Exception {
                if (cd != null) {
                    cd.countDown();
                }

            }

            @Override
            public void handleDataChange(String dataPath, Object data) throws Exception {

            }
        };

        System.out.println("+++++beforePath= "+beforePath);
        this.zkClient.subscribeDataChanges(beforePath, listener);
        if (this.zkClient.exists(beforePath)) {
            cd = new CountDownLatch(1);
            try {
                cd.await();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        this.zkClient.unsubscribeDataChanges(beforePath, listener);

    }

    /**
     * @description: 非阻塞加锁
     * @param {type}
     * @return:
     */
    @Override
    public boolean tryLock() {
        if (currentPath == null || currentPath.length() < 0) {
            currentPath = this.zkClient.createEphemeralSequential("/" + LOCK_PATH + "/window",
                    Thread.currentThread().getName());
            System.out.println("------------>" + currentPath);
        }
        // 获取所有临时节点排序
        List<String> nodesList = this.zkClient.getChildren("/"+LOCK_PATH);
        Collections.sort(nodesList);
        // 如果当前节点在所有节点中排名第一就获取锁成功
        if (currentPath.equals("/"+LOCK_PATH + "/" + nodesList.get(0))) {
            return true;
        } else {
            // 如果当前节点在所有节点中排名不是排名第一，就获取前面的节点，并赋值给beforePath
            for (int i = 0; i < nodesList.size(); i++) {
                if (currentPath.equals("/" + LOCK_PATH +"/"+ nodesList.get(i))) {
                    beforePath = "/"+LOCK_PATH + "/" + nodesList.get(i - 1);
                    return false;
                }
            }

        }

        return false;
    }

    @Override
    public void unlock() {
        // 删除当前临时节点
        zkClient.delete(currentPath);
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return false;
    }

    @Override
    public Condition newCondition() {
        return null;
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {

    }

}
```



## 应用场景类

```java
package com.example.lockdemo;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2019-05-13 18:08:07
 * @LastEditTime: 2019-05-13 20:44:25
 * @LastEditors: 麦子
 */

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.PrintStream;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class TicketDemo implements Runnable {

    private int count = 0;
    // private Lock lock = new ReentrantLock();

    @Override
    public void run() {

        while (count < 100) {
            ZkLockDemo lock = new ZkLockDemo();
            lock.lock();
            try {
                if (count < 100) {
                    ++count;
                    System.out.println(Thread.currentThread().getName() + ": 卖出了第 " + count + " 张票");
                }
            } catch (Exception e) {
                // TODO: handle exception
            } finally {
                lock.unlock();
            }
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }

        }
    }

    public static void main(String[] args) {
        TicketDemo ticketDemo = new TicketDemo();
        Thread threadA = new Thread(ticketDemo, "窗口A");
        Thread threadB = new Thread(ticketDemo, "窗口B");
        Thread threadC = new Thread(ticketDemo, "窗口C");
  

        threadA.start();
        threadB.start();
        threadC.start();
        
        ticketDemo.printOut();


    }

   /***
      控制台日志输入到 文本中
   */
    private void printOut() {
        File f = new File("out.log");
        try {
            f.createNewFile();
            FileOutputStream fileOutputStream = new FileOutputStream(f);
            PrintStream printStream = new PrintStream(fileOutputStream);
            System.setOut(printStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}
```



## 运行结果如下

```java
窗口C: 卖出了第 85 张票
窗口B: 卖出了第 86 张票
窗口A: 卖出了第 87 张票
窗口C: 卖出了第 88 张票
窗口B: 卖出了第 89 张票
窗口A: 卖出了第 90 张票
窗口C: 卖出了第 91 张票
窗口B: 卖出了第 92 张票
窗口A: 卖出了第 93 张票
窗口C: 卖出了第 94 张票
窗口B: 卖出了第 95 张票
窗口A: 卖出了第 96 张票
窗口C: 卖出了第 97 张票
窗口B: 卖出了第 98 张票
窗口A: 卖出了第 99 张票
窗口C: 卖出了第 100 张票
```


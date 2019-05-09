---
title:       "Zookeeper-监听服务动态上下线"
subtitle:    ""
description: ""
date:        2019-05-10
author:      ""
image:       ""
tags:        ["Zookeeper", "服务监听", "zk-java-api用法"]
categories:  ["Tech" ]
---

[TOC]

**说明： 本学习主要对尚硅谷大数据研发部《zookeeper视频》记录，一下文字来源这个视频PPT**

# 公共端代码



```java
package com.example.demo;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @LastEditors: 麦子
 * @Date: 2019-05-09 23:04:35
 * @LastEditTime: 2019-05-09 23:28:19
 */

import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;

public class ZookeeperTemplete {

    private String connec = "127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183";
    private ZooKeeper zooKeeper = null;

     /**
     * @description: 连接服务端
     * @param {type}
     * @return:
     */
    public ZooKeeper getConnection(Watcher watcher) throws Exception {

        zooKeeper = new ZooKeeper(connec, 2000,watcher);
        return zooKeeper;
    }
    
}
```



# 服务器注册处理



```java
package com.example.demo;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @LastEditors: 麦子
 * @Date: 2019-05-09 22:48:56
 * @LastEditTime: 2019-05-10 02:00:07
 */

import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.ZooDefs.Ids;
import org.apache.zookeeper.data.Stat;

public class ServiceHandle implements Watcher {

    public static void main(String[] args) throws Exception {
        ServiceHandle serviceHandle = new ServiceHandle();
        // serviceHandle.regist("tomcat7:2181");
        serviceHandle.regist("tomcat7:2182");
        // serviceHandle.regist("tomcat7:2183");
        serviceHandle.business();
    }

    /**
     * @description: 创建服务节点
     * @param {type}
     * @return:
     * @throws Exception
     */
    public void regist(String hostName) throws Exception {
        try {
            ZookeeperTemplete zookeeperTemplete = new ZookeeperTemplete();
            ZooKeeper zooKeeper = zookeeperTemplete.getConnection(this);

            Stat stat = zooKeeper.exists("/servicelist", false);
            if (stat == null) {
                zooKeeper.create("/servicelist", null, Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            }
            zooKeeper.create("/servicelist/service", hostName.getBytes(), Ids.OPEN_ACL_UNSAFE,
                    CreateMode.EPHEMERAL_SEQUENTIAL);
        } catch (KeeperException | InterruptedException e) {
            e.printStackTrace();
        }
    }

    /**
     * @description: 处理业务逻辑,让服务一直在运行状态模拟
     * @param {type}
     * @return:
     */
    private void business() {
        try {
            Thread.sleep(Long.MAX_VALUE);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    /**
     * @description:  服务端监听不做处理
     * @param {type} 
     * @return: 
     */
    @Override
    public void process(WatchedEvent event) {

    }

}
```



# 客户端监听处理



```java
package com.example.demo;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @LastEditors: 麦子
 * @Date: 2019-05-09 23:11:22
 * @LastEditTime: 2019-05-10 02:19:36
 */

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.PrintStream;
import java.util.ArrayList;
import java.util.List;

import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;

public class ClientHandle implements Watcher {

    private ZooKeeper zooKeeper;

    public static void main(String[] args) throws Exception {
        ClientHandle clientHandle = new ClientHandle();
        clientHandle.getConnection();
        clientHandle.getClientList();
        clientHandle.printOut();
        clientHandle.business();
        
    }

    /**
     * @description: 输出到控制台
     * @param {type} 
     * @return: 
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


    /**
     * @description:  获取连接
     * @param {type} 
     * @return: 
     */
    private void getConnection() throws Exception{
        ZookeeperTemplete zookeeperTemplete = new ZookeeperTemplete();
        zooKeeper = zookeeperTemplete.getConnection(this);
    }

    /**
     * @description: 获取在线服务器列表
     * @param {type}
     * @return:
     */
    private void getClientList() throws Exception {
        List<String> childs = zooKeeper.getChildren("/servicelist", true);
        List<String> hostNameList = new ArrayList<String>();
        for (String nodes : childs) {
            byte[] bytes = zooKeeper.getData("/servicelist/" + nodes, false, null);
            String hostName = new String(bytes);
            hostNameList.add(hostName);
        }
        System.out.println("线上服务器列表: " + hostNameList);
    }

    /**
     * @description: 让客户端端一直执行，模拟
     * @param {type}
     * @return:
     */
    private void business() {
        try {
            Thread.sleep(Long.MAX_VALUE);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    /**
     * @description: 最服务器节点进行监听，监听后，在进行注册，然后在监听。 避免监听一次后不在监听
     * @param {type}
     * @return:
     */
    @Override
    public void process(WatchedEvent event) {
        try {
            getClientList();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```


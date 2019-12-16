---
title:       "Future-设计模式"
subtitle:    ""
description: ""
date:        2019-12-05
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["多线程和高并发", "Future", "设计模式"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://www.cnblogs.com/liang1101/p/6476698.html**

# Future模式

Future模式的核心在于：去除了主函数的等待时间，并使得原本需要等待的时间段可以用于处理其他业务逻辑。

Future模式有点类似于商品订单。在网上购物时，提交订单后，在收货的这段时间里无需一直在家里等候，可以先干别的事情。类推到程序设计中时，当提交请求时，期望得到答复时，如果这个答复可能很慢。传统的是一直持续等待直到这个答复收到之后再去做别的事情，但如果利用Future模式，其调用方式改为异步，而原先等待返回的时间段，在主调用函数中，则可以用于处理其他事务。

总结：就是一个请求需要很长时间， 在这个时间里面不是一直等他的结果，而是去做其他的事情。等那边很慢的请求已经有数据了， 回调给现在的客户端。 

# Data

```java
public interface Data {
    String getResult() throws InterruptedException;
}
```

# FutureData

```java
public class FutureData implements Data {
    RealData realData = null; // FutureData是RealData的封装
    boolean isReady = false; // 是否已经准备好
    private FutureTaskCallBack callback;

    public synchronized void setRealData(RealData realData) {
        if (isReady)
            return;
        this.realData = realData;
        isReady = true;
        notifyAll(); // RealData已经被注入到FutureData中了，通知getResult()方法
    }

    @Override
    public synchronized String getResult() throws InterruptedException {
        if (!isReady) {
            wait(); // 一直等到RealData注入到FutureData中
        }
        return realData.getResult();
    }

    public  synchronized String getCallBackResult() throws InterruptedException {
        if (!isReady) {
            wait(); // 一直等到RealData注入到FutureData中
        }
        return callback.messageCallback(this.realData);
    }

    /**
     * @param callback the callback to set
     */
    public void setCallback(FutureTaskCallBack callback) {
        this.callback = callback;
    }


}
```

# RealData

```java
public class RealData implements Data {
    protected String data;
  
    public RealData handleBusiness(String data){
        try {
            System.out.println(Thread.currentThread().getName()+" 长时间业务处理中.......");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.data = data;
        return this;
    }


    @Override
    public String getResult() throws InterruptedException {
        return data;
    }
}
```

# FutureTaskCallBack

```java
public interface FutureTaskCallBack {
    
    String messageCallback(RealData realData);

}
```



# Client

```java
public class Client {
    public Data request(final String string) {
        final FutureData futureData = new FutureData();

        new Thread(new Runnable() {
            @Override
            public void run() {
                // RealData的构建很慢，所以放在单独的线程中运行
                RealData realData = new RealData();
                realData.handleBusiness(string);
                futureData.setRealData(realData);
            }
        }).start();
        return futureData; // 先直接返回FutureData
    }
}
```

# Main

```java
public class Main {
    public static void main(final String[] args) {
        final Client client = new Client();
        // 这里会立即返回，因为获取的是FutureData，而非RealData
        final Data data = client.request("name=maizi");
        final FutureData future = (FutureData) data;
        future.setCallback(new FutureTaskCallBack() {
            @Override
            public String messageCallback(final RealData realData) {
                System.out.println("回调数据--->" + realData.data);
                return realData.data;
            }

        });
        System.out.println("请求完毕");
        // 使用真实数据
        try {
            // System.out.println("数据=" + data.getResult());

            // 回调
            future.getCallBackResult();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

# 运行结果

```java
请求完毕
Thread-0 长时间业务处理中.......
回调数据--->name=maizi
```



# jdk自带FutureTask示例

## FutureTask的实现

FutureTask是一个基于AQS同步队列实现的一个自定义同步组件，通过对同步状态state的竞争实现acquire或者release操作。
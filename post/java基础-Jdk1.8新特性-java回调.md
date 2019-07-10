---
title:       "java回调"
subtitle:    ""
description: ""
date:        2019-07-04
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "java回调"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://www.jb51.net/article/110015.htm**

# 回调

## 回调的概念

举个例子就是，我们想要问别人一道题，我们把题跟对方说了一下，对方说好，等我做完这道题，我就告诉你，这个时候就用到了回调，因为我们并不知道对方什么时候会做完，而是对方做完了来主动找我们。

## 同步回调

代码运行到某一个位置的时候，如果遇到了需要回调的代码，会在这里等待，等待回调结果返回后再继续执行。

## 异步回调

代码执行到需要回调的代码的时候，并不会停下来，而是继续执行，当然可能过一会回调的结果会返回回来。

# 具体代码

总体的代码还是很简单的，就是模拟了一个打印机，还有一个人，打印机具有打印的功能，但是打印需要时间，不能在收到任务的同时给出反馈，需要等待一段时间才能给出反馈。这个人想做的就是打印一份简历，然后知道打印的结果。这里面代码实现了这两种方式。

```java
public interface Callback {
    void printFinished(String msg);
}
```



```java
public class People {
    Printer printer = new Printer();

    /*
     * 同步回调
     */
    public void goToPrintSyn(Callback callback, String text) {
        printer.print(callback, text);
    }

    /*
     * 异步回调
     */
    public void goToPrintASyn(Callback callback, String text) {
        new Thread(new Runnable() {
            public void run() {
                printer.print(callback, text);
            }
        }).start();
    }
}
```



```java
public class Printer implements Callback {
    public void print(Callback callback, String text) {
        System.out.println("正在打印 . . . ");
        try {
            Thread.currentThread();
            Thread.sleep(3000);// 毫秒
        } catch (Exception e) {
        }
        callback.printFinished("打印完成");
    }

    @Override
    public void printFinished(String msg) {
        System.out.println("打印机告诉我的消息是 ---> " + msg);
    }

    public static void main(String[] args) {
        People people = new People();
        Printer printer = new Printer();
        System.out.println("需要打印的内容是 ---> " + "打印一份简历");
        // people.goToPrintSyn(callback, "打印一份简历");
        people.goToPrintASyn(printer, "打印一份简历");
        System.out.println("我在等待 打印机 给我反馈");
    }

}
```



# 回调和观察者模式区别

**转载地址：https://zhidao.baidu.com/question/111585686.html**

**接口回调：** 是Java多态的一种体现, 可以把使用某一接口的实现类创建的对象的引用, 赋给该接口声明的接口变量中, 那么该接口变量就可以调用被实现的接口中的方法, 当接口变量调用被类实现的接口中的方法时，就是通知相应的对象调用接口的方法。

**观察者模式:**  是将观察者和被观察的对象分离开, 当被观察的对象产生一定变化的时候, 观察者就会根据哪里产生的变化, 产生了变化, 而进行相应的处理.

**备注：**大部分观察着模式是用接口回调的方法来实现的.前者是一种体现, 后者是一种用前者实现的模式, 相当于后者调用前者。
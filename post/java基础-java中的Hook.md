---
title:       "java中的Hook"
subtitle:    ""
description: ""
date:        2020-03-01
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "java中的Hook"]
categories:  ["Tech" ]
---

**转载地址：https://blog.csdn.net/qiaziliping/article/details/73551501?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task**

# 用途

１.  应用程序正常退出，在退出时执行特定的业务逻辑，或者关闭资源等操作。
２.  虚拟机非正常退出，比如用户按下ctrl+c、OutofMemory宕机、操作系统关闭等。在退出时执行必要的挽救措施。

# 示例

感觉有点向回调机制。 

```java
public class App {
    public static void main(String[] args) throws Exception {

        Thread thread1 = new Thread() {
            @Override
            public void run() {
                System.out.println("thread1thread1thread1");
            }
        };

        Thread thread2 = new Thread() {
            @Override
            public void run() {
                System.out.println("thread2thread2thread2");
            }
        };

        // 定义关闭线程
        Thread shutdownThread = new Thread() {
            public void run() {
                System.out.println("shutdownThread...我是钩子");
            }
        };

        thread1.start();
        thread2.start();
        Runtime.getRuntime().addShutdownHook(shutdownThread);
    }
}
```


这个方法的意思就是在jvm中增加一个关闭的钩子，当jvm关闭的时候，会执行系统中已经设置的所有通过方法addShutdownHook添加的钩子，当系统执行完这些钩子后，jvm才会关闭。所以这些钩子可以在jvm关闭的时候进行内存清理、对象销毁等操作。**JVM关闭的时候才执行java钩子，最后执行**

# 运行结果

```java
thread1thread1thread1
thread2thread2thread2
shutdownThread...我是钩子
```


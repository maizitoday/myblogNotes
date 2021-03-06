---
title:       "juc-线程池框架类"
subtitle:    ""
description: ""
date:        2019-12-15
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["多线程和高并发", "java基础"]
categories:  ["Tech" ]
---

[TOC]

# Executor框架

转载地址：https://blog.csdn.net/tongdanping/article/details/79604637

## 概述

我们知道线程池就是线程的集合，线程池集中管理线程，以实现线程的重用，降低资源消耗，提高响应速度等。线程用于执行异步任务，单个的线程既是工作单元也是执行机制，从JDK1.5开始，为了把工作单元与执行机制分离开，Executor框架诞生了，他是一个用于统一创建与运行的接口。Executor框架实现的就是线程池的功能。

## 好处

也就是说，每次我们需要使用线程的时候，可以通过ExecutorService获得线程。它可以有效控制最大并发线程数，提高系统资源的使用率，同时避免过多资源竞争，避免堵塞，同时提供定时执行、定期执行、单线程、并发数控制等功能，也不用使用TimerTask了。

## 框架结构图解

Executor框架包括3大部分

1）任务。也就是工作单元，包括被执行任务需要实现的接口：Runnable接口或者Callable接口；

2）任务的执行。也就是把任务分派给多个线程的执行机制，包括Executor接口及继承自Executor接口的ExecutorService接口。

3）异步计算的结果。包括Future接口及实现了Future接口的FutureTask类。(这是一种设计模式，前面这个设计模式)

## 框架的成员关系图

![20180319221031756](/img/20180319221031756.jpeg)

```java
java.util.concurrent.Executor 负责线程的使用和调度的根接口
		|--ExecutorService 子接口： 线程池的主要接口
				|--ThreadPoolExecutor 线程池的实现类
				|--ScheduledExceutorService 子接口： 负责线程的调度
					|--ScheduledThreadPoolExecutor : 继承ThreadPoolExecutor，实现了ScheduledExecutorService
```



## 框架的使用示意图

![20180319222418739](/img/20180319222418739.jpeg)

1. 创建Runnable并重写run（）方法或者Callable对象并重写call（）方法：
2. 创建Executor接口的实现类ThreadPoolExecutor类或者ScheduledThreadPoolExecutor类的对象，然后调用其execute（）方法或者submit（）方法把工作任务添加到线程中，如果有返回值则返回Future对象。其中Callable对象有返回值，因此使用submit（）方法；而Runnable可以使用execute（）方法，此外还可以使用submit（）方法，只要使用callable（Runnable task）或者callable(Runnable task,  Object result)方法把Runnable对象包装起来就可以，使用callable（Runnable task）方法返回的null，使用callable(Runnable task,  Object result)方法返回result。

## 框架成员

ThreadPoolExecutor实现类、ScheduledThreadPoolExecutor实现类、Future接口、Runnable和Callable接口、Executors工厂类

![20180318215737261](/img/20180318215737261.jpeg)



### ExecutorService

转载地址：https://blog.csdn.net/fwt336/article/details/81530581

ExecutorService是一个接口，提供了管理终止的方法，以及可为跟踪一个或多个异步任务执行状况而生成Future 的方法。

####  创建方式

所有线程池最终都是通过这个方法来创建的。

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)


```

##### corePoolSize

核心线程数，一旦创建将不会再释放。如果创建的线程数还没有达到指定的核心线程数量，将会继续创建新的核心线程，直到达到最大核心线程数后，核心线程数将不在增加；如果没有空闲的核心线程，同时又未达到最大线程数，则将继续创建非核心线程；如果核心线程数等于最大线程数，则当核心线程都处于激活状态时，任务将被挂起，等待空闲线程来执行。

##### maximumPoolSize

最大线程数，允许创建的最大线程数量。如果最大线程数等于核心线程数，则无法创建非核心线程；如果非核心线程处于空闲时，超过设置的空闲时间，则将被回收，释放占用的资源。

##### keepAliveTime

也就是当线程空闲时，所允许保存的最大时间，超过这个时间，线程将被释放销毁，但只针对于非核心线程。

##### unit

时间单位，TimeUnit.SECONDS等。

##### workQueue

任务队列，存储暂时无法执行的任务，等待空闲线程来执行任务。

##### threadFactory

线程工程，用于创建线程。

##### handler

当线程边界和队列容量已经达到最大时，用于处理阻塞时的程序。

#### 线程池的类型

##### 1. 可缓存线程池

```java
ExecutorService cachePool = Executors.newCachedThreadPool();
```

**应用场景**：该线程池比较适合没有固定大小并且比较快速就能完成的小任务，它将为每个任务创建一个线程。那这样子它与直接创建线程对象（new Thread()）有什么区别呢？看到它的第三个参数60L和第四个参数TimeUnit.SECONDS了吗？好处就在于60秒内能够重用已创建的线程。**线程池为无限大**，当执行第二个任务时第一个任务已经完成，会复用执行第一个任务的线程，而不用每次新建线程。

```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

通过它的创建方式可以知道，创建的都是非核心线程，而且最大线程数为Interge的最大值，空闲线程存活时间是1分钟。如果有大量耗时的任务，则不适该创建方式。它只适用于生命周期短的任务。

##### 2. 单线程池

```java
ExecutorService singlePool = Executors.newSingleThreadExecutor();
```

顾名思义，也就是创建一个核心线程：

应用场景：SingleThreadExecutor是只有一个线程的线程池，常用于需要让线程顺序执行，并且在任意时间，只能有一个任务被执行，而不能有多个线程同时执行的场景。

```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }


```

只用一个线程来执行任务，保证任务按FIFO顺序一个个执行。

##### 3. 固定线程数线程池

```java
Executors.newFixedThreadPool(3);		
```

 **应用场景**：适用于为了满足资源管理的需求，而需要适当限制当前线程数量的情景，适用于负载比较重的服务器。

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

```

也就是创建固定数量的可复用的线程数，来执行任务。当线程数达到最大核心线程数，则加入队列等待有空闲线程时再执行。

##### 4. 固定线程数，支持定时和周期性任务

```java
ExecutorService scheduledPool = Executors.newScheduledThreadPool(5);
```

```java
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE,
          DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
          new DelayedWorkQueue());
}
```

可用于替代handler.postDelay和Timer定时器等延时和周期性任务。

```java
public ScheduledFuture<?> schedule(Runnable command,
                                       long delay, TimeUnit unit);

```

```java
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                  long initialDelay,
                                                  long period,
                                                  TimeUnit unit);
```

```java
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                     long initialDelay,
                                                     long delay,
                                                     TimeUnit unit);
```

scheduleAtFixedRate和sheduleWithFixedDelay有什么不同呢？

scheduleAtFixedRate：是以固定频率来执行线程任务，固定频率的含义就是可能设定的固定时间不足以完成线程任务，但是它不管，达到设定的延迟时间了就要执行下一次了。

scheduleWithFixedDelay: 从字面意义上可以理解为就是以固定延迟（时间）来执行线程任务，它实际上是不管线程任务的执行时间的，每次都要把任务执行完成后再延迟固定时间后再执行下一次。

##### 5. 手动创建线程池

```java
private ExecutorService pool = new ThreadPoolExecutor(3, 10,
            10L, TimeUnit.SECONDS,
            new LinkedBlockingQueue<Runnable>(512), Executors.defaultThreadFactory(),
            new ThreadPoolExecutor.AbortPolicy());
```

可以根据自己的需求创建指定的核心线程数和总线程数。

##### 6. newWorkStealingPool：jdk1.8新增

创建持有足够线程的线程池来支持给定的并行级别，并通过使用多个队列，减少竞争，它需要穿一个并行级别的参数，如果不传，则被设定为默认的CPU数量。

**应用场景：**newWorkStealingPool适合使用在很耗时的操作，但是newWorkStealingPool不是ThreadPoolExecutor的扩展，它是新的线程池类ForkJoinPool的扩展，但是都是在统一的一个Executors类中实现，由于能够合理的使用CPU进行对任务操作（并行操作），所以适合使用在很耗时的任务中：



##### 7. 总结

他们里面的适用场景， 主要是针对他们里面实现的时候， 用了不同的数据队列结构。 



### ThreadPoolExecutor 

#### 常用子类

1. FixedThreadPool：固定线程池

2. SingleThreadExecutor：单线程池

3. CachedThreadPool：缓存线程池



### ScheduledThreadPoolExecutor 

转载地址： https://blog.csdn.net/tongdanping/article/details/79627491

#### 源码

通过查看源码，可以发现ScheduledThreadPoolExecutor的实现主要是通过把任务封装为ScheduledFutureTask来实现。ScheduledThreadPoolExecutor通过它的scheduledAtFixedTime（）方法或者scheduledWithFixedDelay（）方法向阻塞队列添加一个实现了RunnableScheduledFutureTask接口的ScheduledFutureTask类对象。

#### 常用子类

1. ScheduledThreadPoolExecutor 

适用于若干个（固定）线程延时或者定期执行任务，同时为了满足资源管理的需求而需要限制后台线程数量的场景。

```java
ScheduledExecutorService stp = Executors.newScheduledThreadPool(int threadNums);
ScheduledExecutorService stp = Executors.newScheduledThreadPool(int threadNums, ThreadFactory threadFactory);
```



2. SingleThreadScheduledExecutor

适用于需要单个线程延时或者定期的执行任务，同时需要保证各个任务顺序执行的应用场景。

```java
ScheduledExecutorService stse = Executors.newSingleThreadScheduledExecutor(int threadNums);
ScheduledExecutorService stp = Executors.newSingleThreadScheduledExecutor(int threadNums, ThreadFactory threadFactory);
```

### Future/FutureTask

转载地址： https://blog.csdn.net/tongdanping/article/details/79630637 

线程运行时候， 返回结果，采用的事Future的一种设计模式， 回调结果。 具体查看博客 [线程基础]



### Runnable和Callable接口

#### 概述

用于实现线程要执行的工作单元。

#### 区别

转载地址： https://www.cnblogs.com/baichunyu/p/11150762.html

Runnable接口中的run()方法的返回值是void，它做的事情只是纯粹地去执行run()方法中的代码而已；Callable接口中的call()方法是有返回值的，是一个泛型，和Future、FutureTask配合可以用来获取异步执行的结果。

这其实是很有用的一个特性，因为多线程相比单线程更难、更复杂的一个重要原因就是因为多线程充满着未知性，某条线程是否执行了？某条线程执行了多久？某条线程执行的时候我们期望的数据是否已经赋值完毕？无法得知，我们能做的只是等待这条多线程的任务执行完毕而已。而Callable+Future/FutureTask却可以获取多线程运行的结果，可以在等待时间太长没获取到需要的数据的情况下取消该线程的任务，真的是非常有用。



### Executors工厂类

转载地址： https://www.jianshu.com/p/4a3dc492e123 

转载地址：https://my.oschina.net/u/1054538/blog/1619619

提供了常见配置线程池的方法，因为ThreadPoolExecutor的参数众多且意义重大，为了避免配置出错，才有了Executors工厂类。

Executors为Executor，ExecutorService，ScheduledExecutorService，ThreadFactory和Callable类提供了一些工具方法，类似于集合中的Collections类的功能。

#### 常用方法

![2962875-5317a86bc6fc9cb8](/img/2962875-5317a86bc6fc9cb8.png)

#### 代码示例

##### 缓存线程池

```java
public class executorsTest {
    public static void main(String[] args) {
        // 这个地方下面代码进行替换。
        ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
        List<Work> workList = Work.buildData();

        for (int i = 0; i < workList.size(); i++) {
            final int index = i; 
            cachedThreadPool.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println(" currentThread name is " + Thread.currentThread().getName()+"---"+workList.get(index).name+"开始工作");
                }
            });
        }

        cachedThreadPool.shutdown();
    }

}

class Work {

    public String name;

    public Work(String name) {
        this.name = name;
    }

    public void doWork() {
        System.out.println(this.name + " 开始工作 ");
    }

    public static List<Work> buildData() {
        List<Work> data = new ArrayList<Work>();
        for (int i = 0; i < 10; i++) {
            data.add(new Work("小明" + i));
        }
        return data;
    }

}
```

运行结果

```java
currentThread name is pool-1-thread-1---小明0开始工作
 currentThread name is pool-1-thread-2---小明1开始工作
 currentThread name is pool-1-thread-4---小明3开始工作
 currentThread name is pool-1-thread-3---小明2开始工作
 currentThread name is pool-1-thread-5---小明4开始工作
 currentThread name is pool-1-thread-2---小明5开始工作
 currentThread name is pool-1-thread-3---小明8开始工作
 currentThread name is pool-1-thread-5---小明6开始工作
 currentThread name is pool-1-thread-2---小明7开始工作
 currentThread name is pool-1-thread-4---小明9开始工作
```

可以看到执行一些耗时不是很长的线程， 可以发现有重复的线程来执行了。 线程池为无限大，当执行第二个任务时第一个任务已经完成，会复用执行第一个任务的线程，而不用每次新建线程。

##### 固定线程池

使用的Thread对象的数量是有限的,如果提交的任务数量大于限制的最大线程数，那么这些任务讲排队，然后当有一个线程的任务结束之后，将会根据调度策略继续等待执行下一个任务。

```java
 ExecutorService cachedThreadPool = Executors.newFixedThreadPool(2);
```

运行结果

```java
currentThread name is pool-1-thread-1---小明0开始工作
 currentThread name is pool-1-thread-2---小明1开始工作
 currentThread name is pool-1-thread-2---小明3开始工作
 currentThread name is pool-1-thread-1---小明2开始工作
 currentThread name is pool-1-thread-1---小明5开始工作
 currentThread name is pool-1-thread-2---小明4开始工作
 currentThread name is pool-1-thread-1---小明6开始工作
 currentThread name is pool-1-thread-2---小明7开始工作
 currentThread name is pool-1-thread-1---小明8开始工作
 currentThread name is pool-1-thread-2---小明9开始工作
```

##### 单线程池

创建一个单线程的线程池。这个线程池只有一个核心线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。

```java
ExecutorService cachedThreadPool = Executors.newSingleThreadExecutor();   
```

运行结果

```java
currentThread name is pool-1-thread-1---小明0开始工作
 currentThread name is pool-1-thread-1---小明1开始工作
 currentThread name is pool-1-thread-1---小明2开始工作
 currentThread name is pool-1-thread-1---小明3开始工作
 currentThread name is pool-1-thread-1---小明4开始工作
 currentThread name is pool-1-thread-1---小明5开始工作
 currentThread name is pool-1-thread-1---小明6开始工作
 currentThread name is pool-1-thread-1---小明7开始工作
 currentThread name is pool-1-thread-1---小明8开始工作
 currentThread name is pool-1-thread-1---小明9开始工作
```

##### 固定线程，定时，周期任务

```java
// 线程启动后， 延迟3秒在进行 
public class executorsTest {
    public static void main(String[] args) {
        ScheduledExecutorService service = Executors.newScheduledThreadPool(5);
        service.schedule(new Runnable() {
            @Override
            public void run() {
                 System.out.println("thread name is "+Thread.currentThread().getName());
            }
        }, 3, TimeUnit.SECONDS);
    }
}
```

```java
public class executorsTest {
    public static void main(String[] args) {
        ScheduledExecutorService service = Executors.newScheduledThreadPool(5);
        service.scheduleAtFixedRate(new Runnable() {
            public void run() {
                System.out.println("不管你现在这个线程是否执行,我从线程延迟启动1秒后， 每3秒就执行一次");
            }
        }, 1, 3, TimeUnit.SECONDS);
    }
}

pool-1-thread-1不管你现在这个线程是否执行,我从线程延迟启动1秒后， 每3秒就执行一次
pool-1-thread-1不管你现在这个线程是否执行,我从线程延迟启动1秒后， 每3秒就执行一次
pool-1-thread-2不管你现在这个线程是否执行,我从线程延迟启动1秒后， 每3秒就执行一次
pool-1-thread-1不管你现在这个线程是否执行,我从线程延迟启动1秒后， 每3秒就执行一次
pool-1-thread-3不管你现在这个线程是否执行,我从线程延迟启动1秒后， 每3秒就执行一次
```

```java
public class executorsTest {
    public static void main(String[] args) {
        ScheduledExecutorService service = Executors.newScheduledThreadPool(5);
        service.scheduleWithFixedDelay(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3_000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " 我从线程延迟启动1秒后， 等上一个线程执行完了， 我在过3秒来执行");
            }
        }, 1, 3, TimeUnit.SECONDS);
    }
}

pool-1-thread-1 我从线程延迟启动1秒后， 等上一个线程执行完了， 我在过3秒来执行
pool-1-thread-1 我从线程延迟启动1秒后， 等上一个线程执行完了， 我在过3秒来执行
pool-1-thread-2 我从线程延迟启动1秒后， 等上一个线程执行完了， 我在过3秒来执行
```

##### newWorkStealingPool

```java
public class executorsTest {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newWorkStealingPool();
        for (int i = 0; i < 3; i++) {
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName());
                }
            });
        }
    }
}
ForkJoinPool-1-worker-2
ForkJoinPool-1-worker-2
ForkJoinPool-1-worker-1
```

#### 线程返回结果示例 

##### Runnable模式

```java
public class executorsTest {
    public static void main(final String[] args) throws InterruptedException, ExecutionException {
        final ExecutorService service = Executors.newFixedThreadPool(5);
        FutureTask<String> task = (FutureTask<String>) service.submit(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3_000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName());
            }
        }, "线程已经完成");
        System.out.println(task.get());
        service.shutdown();
    }
}

pool-1-thread-1
线程已经完成
```

##### FutureTask模式

```java
public class executorsTest {
    public static void main(final String[] args) throws InterruptedException, ExecutionException {
        ExecutorService service = Executors.newFixedThreadPool(5);

        List<Work> workers = Work.buildData();
        List<FutureTask<String>> tasks = new ArrayList<>();
        for (Work worker : workers){
            FutureTask<String> task = new FutureTask<String>(worker);
            service.submit(task);
            tasks.add(task);
        }
        
        for (FutureTask<String> task : tasks){
             System.out.println(task.get());
        }
        service.shutdown();
    }
}

class Work implements Callable<String> {

    public String name;

    public Work(final String name) {
        this.name = name;
    }

    public Work() {
    }

    public void doWork() {
        System.out.println(this.name + " 开始工作 ");
    }

    public static List<Work> buildData() {
        final List<Work> data = new ArrayList<Work>();
        for (int i = 0; i < 10; i++) {
            data.add(new Work("小明" + i));
        }
        return data;
    }

    public static void name() {
        
    }

    @Override
    public String call() throws Exception {
         System.out.println(Thread.currentThread().getName()+"---"+this.name);
        return this.name+": 任务已经完成";
    }

}
```

运行结果

```java
pool-1-thread-1---小明0
pool-1-thread-4---小明3
pool-1-thread-3---小明2
pool-1-thread-2---小明1
pool-1-thread-5---小明4
小明0: 任务已经完成
小明1: 任务已经完成
pool-1-thread-5---小明9
pool-1-thread-3---小明6
小明2: 任务已经完成
小明3: 任务已经完成
小明4: 任务已经完成
pool-1-thread-1---小明8
pool-1-thread-2---小明7
pool-1-thread-4---小明5
小明5: 任务已经完成
小明6: 任务已经完成
小明7: 任务已经完成
小明8: 任务已经完成
小明9: 任务已经完成
```


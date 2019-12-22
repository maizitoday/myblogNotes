---
title:       "juc-并发类"
subtitle:    ""
description: ""
date:        2019-12-12
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["多线程和高并发",  "java基础"]
categories:  ["Tech" ]
---

[TOC]

# happens-before

**转载地址：https://blog.csdn.net/hxm_Code/article/details/102481204**

虽然jvm和执行系统会对指令进行一定的重排，但也是建立在一些原则上的，并非所有指令都可以随便改变执行位置。这些原则就是Happens-Before原则。Happens-Before可以直译为“先行发生”，但其想表达的更深层的意思是“前面操作的结果对后续操作是可见的”。所以比较正式的说法是：Happens-Before约束了编译器的优化行为，虽然允许编译器优化，但是编译器优化后一定要遵循Happens-Before原则。

在Java内存模型中，happens-before 应该翻译成：前一个操作的结果可以被后续的操作获取。讲白点就是前面一个操作把变量a赋值为1，那后面一个操作肯定能知道a已经变成了1。

我们再来看看为什么需要这几条规则？

因为我们现在电脑都是多CPU,并且都有缓存，导致多线程直接的可见性问题。 所以为了解决多线程的可见性问题，就搞出了happens-before原则，让线程之间遵守这些原则。编译器还会优化我们的语句，所以等于是给了编译器优化的约束。不能让它优化的不知道东南西北了！

Happens-Before原则非常重要，它是判断数据是否存在竞争、线程是否安全的主要一句，依靠这个原则，我们可以解决并发环境下两个操作之间是否存在冲突的所有问题。

# AQS

**转载地址：https://blog.csdn.net/alex_xfboy/article/details/85101402**

AbstractQueuedSynchronizer 类（简写成 AQS）是一个抽象类，它是所有的锁队列管理器的父类，JDK 中的各种形式的锁其内部的队列管理器都继承了这个类，它是 Java 并发世界的核心基石。比如 ReentrantLock、ReadWriteLock、CountDownLatch、Semaphone、ThreadPoolExecutor 内部的队列管理器都是它的子类。这个抽象类暴露了一些抽象方法，每一种锁都需要对这个管理器进行定制。而 JDK 内置的所有并发数据结构都是在这些锁的保护下完成的，它是JDK 多线程高楼大厦的地基。

AQS通过内置的FIFO同步队列来完成资源获取线程的排队工作，如果当前线程获取同步状态失败（锁）时，AQS则会将当前线程以及等待状态等信息构造成一个节点（Node）并将其加入同步队列，同时会阻塞当前线程，当同步状态释放时，则会把节点中的线程唤醒，使其再次尝试获取同步状态。

![721070-20170504110246211-10684485](/img/721.png)

不同的自定义同步器争用共享资源的方式也不同。**自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可**，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。

**AQS中采用了一个状态位state + 一个FIFO的队列的方式，记录了锁的获取，释放等。**

## AQS结构

转载：https://blog.csdn.net/pange1991/article/details/80930394

AQS的实现依赖内部的同步队列（FIFO双向队列）来完成同步状态的管理，假如当前线程获取同步状态失败，AQS会将该线程以及等待状态等信息构造成一个Node，并将其加入同步队列，同时阻塞当前线程。当同步状态释放时，唤醒队列的首节点。队列初始化时，头节点head是创建的空Node：new Node() 

## 状态state

AQS使用一个简单的原子int值为大多数同步器表示状态，子类必须定义更改此状态的受保护方法，并定义哪种状态对于此对象意味着被获取或被释放。在互斥锁中它表示着线程是否已经获取了锁，0未获取，1已经获取了，大于1表示重入数。

1. AQS提供了getState()和setState()方法获取/设置状态，是线程可见性的
2. AQS提供了compareAndSetState()方法以原子方式更新state值

```java
private volatile int state;
protected final int getState() {
  return state;
 }
 
 protected final void setState(int newState) {
     state = newState;
 }
 protected final boolean compareAndSetState(int expect, int update) {
     return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
 }
 进行CAS算法。 
```

## 队列元素Node

AQS实现依赖一个FIFO队列，队列元素为Node，Node保存着线程引用和线程状态

元素Node

```java
static final class Node {
    // 当前Node的状态
    volatile int waitStatus; 
 
    // 前驱节点
    volatile Node prev;
 
    // 后继节点
    volatile Node next;
 
    volatile Thread thread;
}
```



# 常用并发类

## CountDownLatch

### 概念

- countDownLatch这个类使一个线程等待其他线程各自执行完毕后再执行。
- 是通过一个计数器来实现的，计数器的初始值是线程的数量。每当一个线程执行完毕后，计数器的值就-1，当计数器的值为0时，表示所有线程都执行完毕，然后在闭锁上等待的线程就可以恢复工作了。

### 源码

countDownLatch类中只提供了一个构造器：

```java
//参数count为计数值
public CountDownLatch(int count) {  };  
```

类中有三个方法是最重要的

```java
//调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public void await() throws InterruptedException { };   
//和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  
//将count值减1
public void countDown() { };  
```

### 示例

```java
public class countDownLatchTest {
    public static void main(String[] args) {

        final CountDownLatch latch = new CountDownLatch(2);
        System.out.println("主线程开始执行…… ……");
        
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + " --- start ");
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " ---  end ");
                latch.countDown();
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + " ==== start ");
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "  ==== end ");
                latch.countDown();
            }
        }).start();

        try {
            latch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("两天线程运行完成。");

    }
}
```

运行结果

```java
主线程开始执行…… ……
Thread-0 --- start 
Thread-1 ==== start 
Thread-1  ==== end 
Thread-0 ---  end 
两天线程运行完成。
```

### 注意

 不是latch.wait()方法， 是latch.await()方法。



## CyclicBarrier

### 概念

转载地址：链接：https://www.jianshu.com/p/333fd8faa56e

它的作用就是会让所有线程都等待完成后才会继续下一步行动。

举个例子，就像生活中我们会约朋友们到某个餐厅一起吃饭，有些朋友可能会早到，有些朋友可能会晚到，但是这个餐厅规定必须等到所有人到齐之后才会让我们进去。这里的朋友们就是各个线程，餐厅就是 CyclicBarrier。

###  源码

```java
public CyclicBarrier(int parties)
public CyclicBarrier(int parties, Runnable barrierAction)
```

- parties 是参与线程的个数
- 第二个构造方法有一个 Runnable 参数，这个参数的意思是最后一个到达线程要做的任务，也就是最后完成后一个回调处理。 

重要的方法

```java
public int await() throws InterruptedException, BrokenBarrierException
public int await(long timeout, TimeUnit unit) throws InterruptedException, BrokenBarrierException, TimeoutException
```

- 线程调用 await() 表示自己已经到达栅栏
- BrokenBarrierException 表示栅栏已经被破坏，破坏的原因可能是其中一个线程 await() 时被中断或者超时

### 示例

栅栏则是所有线程相互等待，直到所有线程都到达某一点时才打开栅栏，然后线程可以继续执行。

```java
public class CyclicBarrierTest extends Thread {

    public static void main(String[] args) {

        CyclicBarrier barrier = new CyclicBarrier(2, new Runnable() {

            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + " 完成最后任务");
            }
        });

        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName()+" 执行A计划");
                try {
                    barrier.await();
                    System.out.println(Thread.currentThread().getName()+"疯狂执行A计划中.......");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName()+" 执行B计划");
                try {
                    barrier.await();
                    System.out.println(Thread.currentThread().getName()+"疯狂执行B计划中.......");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
运行结果：
Thread-0 执行A计划
Thread-1 执行B计划
Thread-1 完成最后任务
Thread-1疯狂执行B计划中.......
Thread-0疯狂执行A计划中.......
```

### 注意

所有等的都是在await()方法前面的， 如果await方法前面的大家都到期了，那么回调就可以执行了。 执行完后， await后面的又都可以各自去执行了。 

## CountDownLatch和CyclicBarrier区别

1. countDownLatch是一个计数器，线程完成一个记录一个，计数器递减，只能只用一次
2. CyclicBarrier的计数器更像一个阀门，需要所有线程都到达，然后继续执行，计数器递增，**提供reset功能，可以多次使用**

## Exchanger

转载地址：https://www.jianshu.com/p/990ae2ab1ae0

### 概念

简单说就是一个线程在完成一定的事务后想与另一个线程交换数据，则第一个先拿出数据的线程会一直等待第二个线程，直到第二个线程拿着数据到来时才能彼此交换对应数据。其定义为 `Exchanger<V>` 泛型类型，其中 V 表示可交换的数据类型。

当一个线程到达 exchange 调用点时，如果其他线程此前已经调用了此方法，则其他线程会被调度唤醒并与之进行对象交换，然后各自返回；如果其他线程还没到达交换点，则当前线程会被挂起，直至其他线程到达才会完成交换并正常返回，或者当前线程被中断或超时返回。

### 源码

```java
Exchanger()：无参构造方法。
```

重要方法

```java
V exchange(V v)：等待另一个线程到达此交换点（除非当前线程被中断），然后将给定的对象传送给该线程，并接收该线程的对象。
```

```
V exchange(V v, long timeout, TimeUnit unit)：等待另一个线程到达此交换点（除非当前线程被中断或超出了指定的等待时间），然后将给定的对象传送给该线程，并接收该线程的对象。
```

### 示例

```java
public class ExchangerTest {
    public static void main(String[] args) throws InterruptedException {
        Exchanger<String> exchanger = new Exchanger<String>();

        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    String temp = exchanger.exchange("张三");
                    System.out.println("resultA -> " + temp);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep(6);
                    String temp = exchanger.exchange("李四");
                    System.out.println("resultB -> " + temp);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });


        Thread t3 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    String temp = exchanger.exchange("王五");
                    System.out.println("resultC -> " + temp);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });


        Thread t4 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    String temp = exchanger.exchange("小明");
                    System.out.println("resultD -> " + temp);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });


        t1.start();
        t2.start();
        t3.start();
        t4.start();
        t1.join();
        t2.join();
        t3.join();
        t4.join();
        System.out.println("等待exchanger跑完");
    }

}
```

运行结果如下：

```java
resultC -> 张三
resultA -> 王五
resultD -> 李四
resultB -> 小明
等待exchanger跑完
```

### 注意

必须是成对的出现， 不然一方总是在等待另一个线程中， 如果一条线程已经运行完成， 那么另外一条线程谁运行的快就和谁进行数据的交换，这种也是一种线程和线程之间的通信。 

## Semaphore

转载地址： https://www.cnblogs.com/mengchunchen/p/9890076.html

转载地址：https://www.jianshu.com/p/0572483ecec7

### 概念

Semaphore也叫信号量，在JDK1.5被引入，用来控制同时访问某个特定资源的操作数量，或者同时执行某个指定操作的数量。执行一段代码的时候， 允许几条线程同时进入进来。 访问特定资源前，必须使用acquire方法获得许可，如果许可数量为0，该线程则一直阻塞，直到有可用许可。访问资源后，使用release释放许可。

比如模拟一个停车场停车信号，假设停车场只有两个车位，一开始两个车位都是空的。这时同时来了两辆车，看门人允许它们进入停车场，然后放下车拦。以后来的车必须在入口等待，直到停车场中有车辆离开。这时，如果有一辆车离开停车场，看门人得知后，打开车拦，放入一辆，如果又离开一辆，则又可以放入一辆，**如此往复**。

**Semaphore 管理一系列许可证**

1. 每个 `acquire()` 方法阻塞，直到有一个许可证可以获得然后拿走一个许可证。

2. 每个 `release()` 方法增加一个许可证，这可能会释放一个阻塞的 `acquire()` 方法。

3. 不使用实际的许可对象，Semaphore 只对可用许可的号码进行计数，并采取相应的行动。

4. Semaphore 在计数器不为 0 的时候对线程就放行，一旦达到 0，那么所有请求资源的新线程都会被阻塞，包括增加请求到许可的线程，Semaphore 是不可重入的。 

5. 每一次请求一个许可都会导致计数器减少 1，同样每次释放一个许可都会导致计数器增加 1，一旦达到 0，新的许可请求线程将被挂起。

### 实现场景

1. 一个固定长度的资源池，当池为空时，请求资源会失败。使用Semaphore可以实现当池为空时，请求会阻塞，非空时解除阻塞。

2. 也可以使用Semaphore将任何一种容器变成有界阻塞容器。

### 源码

**构造函数**

- 两个构造方法，都必须提供许可的数量，第二个构造方法可以指定是公平模式还是非公平模式，默认非公平模式。

```java
//permits是允许同时运行的线程数目
public Semaphore(int permits) {
        sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

Semaphore 有两种模式，**公平模式** 和 **非公平模式**

- 公平模式就是调用 acquire 的顺序就是获取许可证的顺序，遵循 FIFO。

- 非公平模式是抢占式的，也就是有可能一个新的获取线程恰好在一个许可证释放时得到了这个许可证，而前面还有等待的线程。

  

**常用方法**

```java
// 创建具有给定的许可数和非公平的公平设置的 Semaphore。
Semaphore(int permits)
// 创建具有给定的许可数和给定的公平设置的 Semaphore。
Semaphore(int permits, boolean fair)

// 从此信号量获取一个许可，在提供一个许可前一直将线程阻塞，否则线程被中断。
void acquire()
// 从此信号量获取给定数目的许可，在提供这些许可前一直将线程阻塞，或者线程已被中断。
void acquire(int permits)
// 从此信号量中获取许可，在有可用的许可前将其阻塞。
void acquireUninterruptibly()
// 从此信号量获取给定数目的许可，在提供这些许可前一直将线程阻塞。
void acquireUninterruptibly(int permits)
// 返回此信号量中当前可用的许可数。
int availablePermits()
// 获取并返回立即可用的所有许可。
int drainPermits()
// 返回一个 collection，包含可能等待获取的线程。
protected Collection<Thread> getQueuedThreads()
// 返回正在等待获取的线程的估计数目。
int getQueueLength()
// 查询是否有线程正在等待获取。
boolean hasQueuedThreads()
// 如果此信号量的公平设置为 true，则返回 true。
boolean isFair()
// 根据指定的缩减量减小可用许可的数目。
protected void reducePermits(int reduction)
// 释放一个许可，将其返回给信号量。
void release()
// 释放给定数目的许可，将其返回到信号量。
void release(int permits)
// 返回标识此信号量的字符串，以及信号量的状态。
String toString()
// 仅在调用时此信号量存在一个可用许可，才从信号量获取许可。
boolean tryAcquire()
// 仅在调用时此信号量中有给定数目的许可时，才从此信号量中获取这些许可。
boolean tryAcquire(int permits)
// 如果在给定的等待时间内此信号量有可用的所有许可，并且当前线程未被中断，则从此信号量获取给定数目的许可。
boolean tryAcquire(int permits, long timeout, TimeUnit unit)
// 如果在给定的等待时间内，此信号量有可用的许可并且当前线程未被中断，则从此信号量获取一个许可。
boolean tryAcquire(long timeout, TimeUnit unit)
```

### 示例

```java
public class SemaphoreTest implements Runnable {

    private Semaphore semaphore;

    public SemaphoreTest(Semaphore semaphore) {
        this.semaphore = semaphore;
    }

    @Override
    public void run() {
        try {
            semaphore.acquire();
            System.out.println(Thread.currentThread().getName() + "开始工作");
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("当前可用的许可证---"+semaphore.availablePermits());
            System.out.println(Thread.currentThread().getName() + "结束工作");
            semaphore.release();
            System.out.println("当前可用的许可证---"+semaphore.availablePermits());
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
    }

    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(1);
        SemaphoreTest test = new SemaphoreTest(semaphore);
        Thread t = new Thread(test);
        Thread t2 = new Thread(test);
        t.start();
        t2.start();

    }

}
```

允许结果

```java
Thread-0开始工作
当前可用的许可证---0
Thread-0结束工作
当前可用的许可证---1
Thread-1开始工作
当前可用的许可证---0
Thread-1结束工作
当前可用的许可证---1
```

### 注意

从上面的列子可以看出， 他可以控制其中一段代码，同时可以进入几条线程，也就可以说是synchronize的加强版本，同时如果设置为只有一个信号量的话， 就是一个Lock的机制了。 他是拿到一个许可证后，然后需要释放这个许可证，然后在可以下个线程进入，不然的话， 许可证没有释放的话，就会让当前线程一直阻塞住。 

## ForkJoin框架

转载地址： https://www.cnblogs.com/linlinismine/p/9295701.html

转载地址： https://blog.csdn.net/weixin_41404773/article/details/80733324

### 概念

fork/join作为一个并发框架在jdk7的时候就加入到了我们的java并发包java.util.concurrent中，并且在java 8 的lambda并行流中充当着底层框架的角色。这样一个优秀的框架设计，我自己想了解一下它的底层代码是如何实现的，所以我尝试的去阅读了JDK相关的源码。下面我打算分享一下阅读完之后的心得~。

#### fork/join的设计思路

了解一个框架的第一件事，就是先了解别人的设计思路!

![801](/img/801.png)

 

fork/join大体的执行过程就如上图所示，先把一个大任务分解(fork)成许多个独立的小任务，然后起多线程并行去处理这些小任务。处理完得到结果后再进行合并(join)就得到我们的最终结果。显而易见的这个框架是借助了现代计算机多核的优势并行去处理数据。这看起来好像没有什么特别之处，这个套路很多人都会，并且工作中也会经常运用~。其实fork/join的最特别之处在于它还运用了一种叫**work-stealing(工作窃取)的算法**，这种算法的设计思路在于把分解出来的小任务放在多个双端队列中，而线程在队列的头和尾部都可获取任务。当有线程把当前负责队列的任务处理完之后，它还可以从那些还没有处理完的队列的尾部窃取任务来处理，这连线程的空余时间也充分利用了！。work-stealing原理图如下：**

 ![802](/img/802.png)

#### 实现fork/join  定义了哪些角色

​       了解设计原理，这仅仅是第一步！要了解别人整个的实现思路， 还需要了解别人为了实现这个框架定义了哪些角色，并了解这些角色的职责范围是什么的。因为知道谁负责了什么，谁做什么，这样整个逻辑才能串起来！在JAVA里面角色是以类的形式定义的,而了解类的行为最直接的方式就是看定义的公共方法~。

**这里介绍JDK里面与fork/join相关的主要几个类：**

##### ForkJoinPool

充当fork/join框架里面的管理者，最原始的任务都要交给它才能处理。它负责控制整个fork/join有多少个workerThread，workerThread的创建，激活都是由它来掌控。它还负责workQueue队列的创建和分配，每当创建一个workerThread，它负责分配相应的workQueue。然后它把接到的活都交给workerThread去处理，它可以说是整个frok/join的容器。

##### ForkJoinWorkerThread

fork/join里面真正干活的"工人"，本质是一个线程。里面有一个ForkJoinPool.WorkQueue的队列存放着它要干的活，接活之前它要向ForkJoinPool注册(registerWorker)，拿到相应的workQueue。然后就从workQueue里面拿任务出来处理。它是依附于ForkJoinPool而存活，如果ForkJoinPool的销毁了,它也会跟着结束。

##### ForkJoinPool.WorkQueue

双端队列就是它，它负责存储接收的任务

##### ForkJoinTask

代表fork/join里面任务类型，我们一般用它的两个子类RecursiveTask、RecursiveAction。这两个区别在于RecursiveTask任务是有返回值，RecursiveAction没有返回值。任务的处理逻辑包括任务的切分都集中在compute()方法里面

### 源码

原文地址：https://www.jianshu.com/p/42e9cd16f705

#### ForkJoinPool 

他就是一个线程池。

以Runtime.getRuntime().availableProcessors()的返回值作为parallelism来创建ForkJoinPool

```java
public ForkJoinPool() ：
```

创建一个包含parallelism个并行线程的ForkJoinPool

```java
public ForkJoinPool(int parallelism)：
```



#### ForkJoinTask<V>

这是一个运行在`ForkJoinPool`中的抽象的任务类。类型`V`指定了任务的返回结果。**ForkJoinTask是一个类似线程的实体**，它表示任务的轻量级抽象，而不是实际的执行线程。

##### fork()

方法提交并执行异步任务，该方法返回`ForkJoinTask`并且调用线程继续运行。

##### join()

方法等待任务直到返回结果。

##### invoke()

方法是组合了fork()和join()，它开始一个任务并等待结束返回结果。

此外，`ForkJoinTask`中还提供了用于一次调用多个任务的两个静态方法

```java
-  **static void invokeAll(ForkJoinTask<?> task1, ForkJoinTask<?> task2)** :执行两个任务
-  **static void invokeAll(ForkJoinTask<?>… taskList)**:执行任务集合
```



#### RecursiveAction

这是一个递归的`ForkJoinTask`子类，不返回结果。`Recursive`意思是任务可以通过分治策略分成自己的子任务。

 

#### 在ForkJoinPool*中执行*ForkJoinTasks

1. <T>T invoke(ForkJoinTask<T> task):执行指定任务并返回结果，该方法是异步的，调用的线程会一直等待直到该方法返回结果，对于*RecursiveAction*任务来说，参数类型是*Void*.

2. void execute(ForkJoinTask<?> task):异步执行指定的任务，调用的线程一直等待知道任务完成才会继续执行。

   

### 示例

转载地址： https://blog.csdn.net/weixin_41404773/article/details/80733324

```java
public class RecursiveTaskDemo extends RecursiveTask<Integer> {
    private static final long serialVersionUID = 1L;
    /**
     * 每个"小任务"最多只打印70个数
     */
    private static final int MAX = 4;
    private int arr[];
    private int start;
    private int end;

    public RecursiveTaskDemo(int[] arr, int start, int end) {
        this.arr = arr;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        int sum = 0;
        // 当end-start的值小于MAX时候，开始打印
        if ((end - start) < MAX) {
            System.err.println(Thread.currentThread().getName() + "=============");
            for (int i = start; i < end; i++) {
                sum += arr[i];
            }
            return sum;
        } else {
            System.err.println(Thread.currentThread().getName() + "=====任务分解======");
            // 将大任务分解成两个小任务
            int middle = (start + end) / 2;
            RecursiveTaskDemo left = new RecursiveTaskDemo(arr, start, middle);
            RecursiveTaskDemo right = new RecursiveTaskDemo(arr, middle, end);
            // 并行执行两个小任务
            left.fork();
            right.fork();
            // 把两个小任务累加的结果合并起来
            return left.join() + right.join();
        }
    }

    public static void main(String[] args) {
        int arr[] = new int[10];

        for (int i = 0; i < 10; i++) {
            arr[i] = i;
        }

        // 创建包含Runtime.getRuntime().availableProcessors()返回值作为个数的并行线程的ForkJoinPool
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        Integer integer = forkJoinPool.invoke(new RecursiveTaskDemo(arr, 0, arr.length));
        System.out.println("计算出来的总和=" + integer);

        // 关闭线程池
        forkJoinPool.shutdown();
    }

}
```

运行结果：

```java
ForkJoinPool-1-worker-1=====任务分解======
ForkJoinPool-1-worker-3=====任务分解======
ForkJoinPool-1-worker-2=====任务分解======
ForkJoinPool-1-worker-1=============
ForkJoinPool-1-worker-1=============
ForkJoinPool-1-worker-1=============
ForkJoinPool-1-worker-3=============
计算出来的总和=45
```

相当于通过fork方法， 重新开启了另一线程，然后再次调用现在的compute的方法，最后在等待join的结果返回，最后把join方法的所有值从队列中一起加起来。 



### 注意

- *ForkJoinPool*中的线程是守护线程，不必显式地关闭池
- 执行一个*ForkJoinTask*既可以通过调用它自己的`invoke()`或`fork()`方法，也可以提交任务给*ForkJoinPool*并调用它的`invoke()`或者`execute()`方法
- 在`ForkJoinTask`中使用`join()`方法，可以合并子任务的结果
- `invoke()`方法会等待子任务完成，但是`execute()`方法不会



## Phaser

转载：https://blog.csdn.net/u010010428/article/details/52178545 

### 概念

Phaser是一个更强大的、更复杂的同步辅助类，可以代替CyclicBarrier CountDownLatch的功能，但是比他们更强大。 Phaser类机制是在每一步结束的位置对线程进行同步，当所有的线程都完成了这一步，才能进行下一步。 

当我们有并发任务并且需要分解成几步执行的时候，这种机制就非常适合。 

### 源码

参加会议所需的聚会人数

```java

public Phaser(int parties) {
  this(null, parties);
}

public Phaser() {
  this(null, 0);
}
```



#### 常用方法

##### 1. 设定任务数

（这是官方解释，简单的讲比如我们指定任务数为3，那3个线程都运行到指定地方他们才会分别继续往下执行）

```java
// 批量设置任务数
public int bulkRegister(int parties)
// 动态增加1个任务数
public int register()
// 减少一个任务数
public int arriveAndDeregister()
```

##### 2. 到达等待

```java
// 线程执行到此开始等待，满足条件(任务数满足)则继续执行，等待其他线程到达完成。 
public int arriveAndAwaitAdvance() 
// 线程执行到此不用等待继续执行，但会让任务数加1，然后会重置任务数。
public int arrive()
// 传入的phase表示翻越几个屏障，如果传入的等于getPhase则等待
public int awaitAdvance(int phase)
// 可中断的awaitAdvance
public int awaitAdvanceInterruptibly(int phase)；
// 可规定时间指定栏数未变，则抛出异常
public int awaitAdvanceInterruptibly(int phase,
                                         long timeout, TimeUnit unit)
```

##### 3. 得到栏数以及任务数

```java
// 当前执行到第几个屏障
public final int getPhase()
// 得到注册的任务数
public int getRegisteredParties()
// 已经执行的任务数
public int getArrivedParties()
// 还未执行的任务数
public int getUnarrivedParties()
```

##### 4. 通过屏障时调用

```java
// 该方法返回true则直接通过屏障 false则屏障继续工作，要想实现自己的逻辑代码需要复写此方法
protected boolean onAdvance(int phase, int registeredParties)
```

##### 5. 关闭屏障

```java
// 取消屏障
public void forceTermination()
// phaser是否销毁
public boolean isTerminated()
```



终止判断

当phaser处于终止状态的时候，arriveAndAwaitAdvance() 和 awaitAdvance() 立即返回一个负数，而不再是一个正值了，如果知道phaser可能会被终止，就需要验证这些方法的值，以确定phaser是不是被终止了。 
被终止的phaser不会保证参与者的同步。

### 示例

转载地址：https://www.jianshu.com/p/1517a379fb91

```java
public class PharseDemo {

    public static void main(String[] args) throws InterruptedException {
        // 10名运动员，每名运动员在同一时间点的开始
        Phaser pharse = new Phaser(5) {
            @Override
            protected boolean onAdvance(int phase, int registeredParties) {
                if (phase == 0) {
                    System.out.println("所有人员已经到赛场");
                } else if (phase == 1) {
                    System.out.println("所有人员准备完毕，开始比赛");
                } else if (phase == 2) {
                    System.out.println("比赛结束");
                }
                return super.onAdvance(phase, registeredParties);
            }
        };
        // 创建10个线程
        Runner run = new Runner(pharse);
        Thread[] thread = new Thread[5];
        for (int i = 0; i < 5; i++) {
            thread[i] = new Thread(run);
        }
        Thread.sleep(3000);
        for (int i = 0; i < 5; i++) {
            thread[i].start();
        }
    }

}

class Runner implements Runnable {

    private Phaser phaser;
    Random random = new Random();

    public Runner(Phaser phaser) {
        super();
        this.phaser = phaser;
    }

    @Override
    public void run() {
        try {
            // 第一阶段 到达赛场 每个运动员情况不一样
            Thread.sleep(random.nextInt(3000));
            System.out.println(Thread.currentThread().getName() + "已经到达赛场");
            phaser.arriveAndAwaitAdvance();
            // phaser.arrive();

            // 第二阶段 开始准备
            Thread.sleep(random.nextInt(3000));
            System.out.println(Thread.currentThread().getName() + "已经准备好");
            phaser.arriveAndAwaitAdvance();

          

            // 第二阶段 到达终点
            Thread.sleep(random.nextInt(3000));
            System.out.println(Thread.currentThread().getName() + "已经到达终点");
            phaser.arriveAndAwaitAdvance();

           
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}
```

运行结果

```java
Thread-2已经到达赛场
Thread-1已经到达赛场
Thread-4已经到达赛场
Thread-0已经到达赛场
Thread-3已经到达赛场
所有人员已经到赛场
Thread-0已经准备好
Thread-1已经准备好
Thread-4已经准备好
Thread-3已经准备好
Thread-2已经准备好
所有人员准备完毕，开始比赛
Thread-4已经到达终点
Thread-2已经到达终点
Thread-1已经到达终点
Thread-3已经到达终点
Thread-0已经到达终点
比赛结束
```

#### 独特的概念是phase

Phaser有一个独特的概念是phase，默认是0，当getRegisteredParties()个parties都到达了，phase即升级为1，依次类推，2、3...。直观上理解，phase每升一次级，相当于Phaser重生，只是phase号加一了而已。如果你在一个线程或多个线程中不断的调用arrive()，那么会发现getArrivedParties()值不断的增长，当长到getRegisteredParties()值时，你会发现phase自动升级了，同时getArrivedParties()恢复为0。



### 注意

1. CyclicBarrier CountDownLatch 只能在构造时指定参与量，而phaser可以动态的增减参与量。
2. phaser有一个重大特性，就是不必对它的方法进行异常处理。置于休眠的线程不会响应中断事件，不会抛出interruptedException异常， 只有一个方法会响应：AwaitAdvanceInterruptibly(int phaser).
3. 模拟代替CountDownLatch功能，只需要当前线程arriveAndAwaitAdvance()之后运行需要的代码之后，就arriveAndDeregister()取消当前线程的注册。
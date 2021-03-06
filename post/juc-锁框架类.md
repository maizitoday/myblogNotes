---
title:       "juc-锁框架类"
subtitle:    ""
description: "Lock,Synchronized,ReadWriteLock,"
date:        2019-12-17
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["多线程和高并发","java基础"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：好文 https://blog.csdn.net/justloveyou_/article/details/54972105**

# 概述

　我们已经知道，synchronized 是java的关键字，是Java的内置特性，在JVM层面实现了对临界资源的同步互斥访问，但 synchronized 粒度有些大，在处理实际问题时存在诸多局限性，比如响应中断等。Lock 提供了比 synchronized更广泛的锁操作，它能以更优雅的方式处理线程同步问题。

# synchronized 的局限性 与 Lock 的优点

如果一个代码块被synchronized关键字修饰，当一个线程获取了对应的锁，并执行该代码块时，其他线程便只能一直等待直至占有锁的线程释放锁。事实上，占有锁的线程释放锁一般会是以下三种情况之一

- 占有锁的线程执行完了该代码块，然后释放对锁的占有；
- 占有锁线程执行发生异常，此时JVM会让线程自动释放锁；
- 占有锁线程进入 WAITING 状态从而释放锁，例如在该线程中调用wait()方法等。

synchronized 是Java语言的内置特性，可以轻松实现对临界资源的同步互斥访问。那么，为什么还会出现Lock呢？试考虑以下三种情况：

## 1. 无法自动释放锁 

在使用synchronized关键字的情形下，假如占有锁的线程由于要等待IO或者其他原因（比如调用sleep方法）被阻塞了，但是又没有释放锁，那么其他线程就只能一直等待，别无他法。这会极大影响程序执行效率。因此，就需要有一种机制可以不让等待的线程一直无期限地等待下去（比如只等待一定的时间 (解决方案：tryLock(long time, TimeUnit unit)) 或者 能够响应中断 (解决方案：lockInterruptibly())），这种情况可以通过 Lock 解决。

## 2. 多线程读也会被锁

我们知道，当多个线程读写文件时，读操作和写操作会发生冲突现象，写操作和写操作也会发生冲突现象，但是读操作和读操作不会发生冲突现象。但是如果采用synchronized关键字实现同步的话，就会导致一个问题，即当多个线程都只是进行读操作时，也只有一个线程在可以进行读操作，其他线程只能等待锁的释放而无法进行读操作。因此，需要一种机制来使得当多个线程都只是进行读操作时，线程之间不会发生冲突。同样地，Lock也可以解决这种情况 (解决方案：ReentrantReadWriteLock) 。

## 3. 无法查看是否获取到锁

我们可以通过Lock得知线程有没有成功获取到锁 **(解决方案：ReentrantLock)** ，但这个是synchronized无法办到的。

## 总结

上面提到的三种情形，我们都可以通过Lock来解决，但 synchronized 关键字却无能为力。事实上，Lock 是 java.util.concurrent.locks包 下的接口，Lock 实现提供了比 synchronized 关键字 更灵活、更广泛、粒度更细 的锁操作，它能以更优雅的方式处理线程同步问题。也就是说，Lock提供了比synchronized更多的功能。但是要注意以下几点：

1) synchronized是Java的关键字，因此是Java的内置特性，是基于JVM层面实现的，其经过编译之后，会在同步块的前后分别形成 monitorenter 和 monitorexit 两个字节码指令；而Lock是一个Java接口，是基于JDK层面实现的，通过这个接口可以实现同步访问；

2) 采用synchronized方式不需要用户去手动释放锁，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁的占用；而 Lock则必须要用户去手动释放锁 (发生异常时，不会自动释放锁)，如果没有主动释放锁，就有可能导致死锁现象。

这是很好理解的。Synchronized方式是Java原生支持的，开发人员在使用它来解决并发问题时，一定会方便很多，在这里，开发人员就不需要手动获取锁和释放锁，这些操作均有Java自身自动完成；而Lock方式是JDK层面的提供给开发人员的接口，因此开发人员在使用它来解决并发问题时，需要手动获取锁和释放锁。

# Lock 和 Synchronized 的选择

总的来说，Lock 和 Synchronized 有以下几点不同：

(1) Lock是一个接口，是JDK层面的实现；而synchronized是Java中的关键字，是Java的内置特性，是JVM层面的实现；

(2) synchronized 在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；

(3) Lock 可以让等待锁的线程响应中断，而使用synchronized时，等待的线程会一直等待下去，不能够响应中断；

(4) 通过Lock可以知道有没有成功获取锁，而synchronized却无法办到；

(5) Lock可以提高多个线程进行读操作的效率。

在性能上来说，如果竞争资源不激烈，两者的性能是差不多的。而当竞争资源非常激烈时（即有大量线程同时竞争），此时Lock的性能要远远优于synchronized。所以说，在具体使用时要根据适当情况选择。

# 锁的相关概念介绍

## 1. 可重入锁

广义上的可重入锁指的是可重复可递归调用的锁，在外层使用锁之后，在内层仍然可以使用，并且不发生死锁（前提得是同一个对象或者class），这样的锁就叫做可重入锁。

举个简单的例子，当一个线程执行到某个synchronized方法时，比如说method1，而在method1中会调用另外一个synchronized方法method2，此时线程不必重新去申请锁，而是可以直接执行方法method2。

```java
class MyClass {
    public synchronized void method1() {
        method2();
    }

    public synchronized void method2() {

    }
}
```

上述代码中的两个方法method1和method2都用synchronized修饰了。假如某一时刻，线程A执行到了method1，此时线程A获取了这个对象的锁，而由于method2也是synchronized方法，假如synchronized不具备可重入性，此时线程A需要重新申请锁。但是，这就会造成死锁，因为线程A已经持有了该对象的锁，而又在申请获取该对象的锁，这样就会线程A一直等待永远不会获取到的锁。而由于synchronized和Lock都具备可重入性，所以不会发生上述现象。



## 2. 可中断锁

顾名思义，可中断锁就是可以响应中断的锁。在Java中，**synchronized就不是可中断锁，而Lock是可中断锁。**

如果某一线程A正在执行锁中的代码，另一线程B正在等待获取该锁，可能由于等待时间过长，线程B不想等待了，想先处理其他事情，我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁。在前面演示tryLock(long time, TimeUnit unit)和lockInterruptibly()的用法时已经体现了Lock的可中断性。(**只能中断在等待的线程**)



## 3. 公平锁

公平锁即 **尽量** 以请求锁的顺序来获取锁。比如，同是有多个线程在等待一个锁，当这个锁被释放时，等待时间最久的线程（最先请求的线程）会获得该所，这种就是公平锁。而非公平锁则无法保证锁的获取是按照请求锁的顺序进行的，这样就可能导致某个或者一些线程永远获取不到锁。

在Java中，**synchronized就是非公平锁（抢占锁）**，它无法保证等待的线程获取锁的顺序。而对于ReentrantLock 和 ReentrantReadWriteLock，它默认情况下是非公平锁，但是可以设置为 **公平锁（协同式线程调度）**。

```java
new ReentrantLock(true); // 公平锁
new ReentrantReadWriteLock(true);

new ReentrantLock(false); // 非公平锁
new ReentrantReadWriteLock(true);
```



## 4. 读写锁

读写锁将对临界资源的访问分成了两个锁，一个读锁和一个写锁。正因为有了读写锁，才使得多个线程之间的读操作不会发生冲突。ReadWriteLock就是读写锁，它是一个接口，ReentrantReadWriteLock实现了这个接口。可以通过readLock()获取读锁，通过writeLock()获取写锁。

# java.util.concurrent.locks包下常用的类与接口

## 包结构图

![20170211102428481](/img/20170211102428481.png)

## Lock

### 源码

```java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;  // 可以响应中断
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;  // 可以响应中断
    void unlock();
    Condition newCondition();
}
```

下面来逐个分析Lock接口中每个方法。lock()、tryLock()、tryLock(long time, TimeUnit unit) 和 lockInterruptibly()都是用来获取锁的。unLock()方法是用来释放锁的。newCondition() 返回 绑定到此 Lock 的新的 Condition 实例 ，**用于线程间的协作**

### 常用方法

#### 1. lock()方法

lock()方法是平常使用得最多的一个方法，就是用来获取锁。**如果锁已被其他线程获取，则进行等待。**在前面已经讲到，如果采用Lock，必须主动去释放锁，并且在发生异常时，不会自动释放锁。因此，一般来说，使用Lock必须在try…catch…块中进行，并且将释放锁的操作放在finally块中进行，以保证锁一定被被释放，防止死锁的发生。

通常使用Lock来进行同步的话，是以下面这种形式去使用的：

```java
Lock lock = ...;
lock.lock();
try{
    //处理任务
}catch(Exception ex){

}finally{
    lock.unlock();   //释放锁
}
```



#### 2. tryLock() & tryLock(time, unit)

tryLock()方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true；如果获取失败（即锁已被其他线程获取），则返回false，也就是说，**这个方法无论如何都会立即返回（在拿不到锁时不会一直在那等待）。**

tryLock(long time, TimeUnit unit)方法和tryLock()方法是类似的，只不过区别在于**这个方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false，同时可以响应中断。**如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。

一般情况下，通过tryLock来获取锁时是这样使用的：

```java
Lock lock = ...;
if(lock.tryLock()) {
     try{
         //处理任务
     }catch(Exception ex){

     }finally{
         lock.unlock();   //释放锁
     } 
}else {
    //如果不能获取锁，则直接做其他事情
}
```

#### 3. lockInterruptibly()

lockInterruptibly()方法比较特殊，当通过这个方法去获取锁时，如果线程 正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。例如，当两个线程同时通过lock.lockInterruptibly()想获取某个锁时，假若此时线程A获取到了锁，而线程B只有在等待，那么对线程B调用threadB.interrupt()方法能够中断线程B的等待过程。

由于lockInterruptibly()的声明中抛出了异常，所以lock.lockInterruptibly()必须放在try块中或者在调用lockInterruptibly()的方法外声明抛出 InterruptedException，但推荐使用后者，原因稍后阐述。

因此，lockInterruptibly()一般的使用形式如下：

```java
public void method() throws InterruptedException {
    lock.lockInterruptibly();
    try {  
     //.....
    }
    finally {
        lock.unlock();
    }  
}
```

注意，当一个线程获取了锁之后，是不会被interrupt()方法中断的。**因为interrupt()方法只能中断阻塞过程中的线程而不能中断正在运行过程中的线程。**因此，当通过lockInterruptibly()方法获取某个锁时，如果不能获取到，那么只有进行等待的情况下，才可以响应中断的。与 synchronized 相比，当一个线程处于等待某个锁的状态，是无法被中断的，只有一直等待下去。



### 注意

在使用Lock时，无论以哪种方式获取锁，习惯上最好一律将获取锁的代码放到 try…catch…，因为我们一般将锁的unlock操作放到finally子句中，如果线程没有获取到锁，在执行finally子句时，就会执行unlock操作，从而抛出 IllegalMonitorStateException，因为该线程并未获得到锁却执行了解锁操作。



### 子类锁---ReentrantLock

#### 概述

ReentrantLock，即可重入锁。**ReentrantLock是唯一实现了Lock接口的类，并且ReentrantLock提供了更多的方法。**

#### 常用方法

```java
isFair() //判断锁是否是公平锁

isLocked() //判断锁是否被任何线程获取了

isHeldByCurrentThread() //判断锁是否被当前线程获取了

hasQueuedThreads() //判断是否有线程在等待该锁

getHoldCount() //查询当前线程占有lock锁的次数

getQueueLength() // 获取正在等待此锁的线程数

getWaitQueueLength(Condition condition) // 获取正在等待此锁相关条件condition的线程数
```

#### 代码示例

```java
public class ReentrantLockTest {

    public static void main(String[] args) {

        ExecutorService service = Executors.newFixedThreadPool(3);
        Lock lock = new ReentrantLock();
        Work work = new Work(lock);
        for (int i = 0; i < 3; i++) {
            service.execute(new Runnable() {
                @Override
                public void run() {
                    work.doWork();
                }
            });
        }

    }
}

class Work {

    private String name;
    private Lock lock;

    public Work(Lock lock) {
        this.lock = lock;
    }

    public void doWork() {
        try {
            lock.lock();
            List<String> names = Arrays.asList("小强", "小米", "小明");
            int index = (int) (Math.random() * names.size());
            this.name = names.get(index);
            System.out.println(this.name + ": 被锁定" + Thread.currentThread().getName());
            System.out.println(this.name + ": 努力工作" + Thread.currentThread().getName());
            TimeUnit.SECONDS.sleep(2);
            System.out.println(this.name + ": 工作已经完成" + Thread.currentThread().getName());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
            System.out.println(this.name + "锁已经释放");
        }
    }

}


小明: 被锁定pool-1-thread-1
小明: 努力工作pool-1-thread-1
小明: 工作已经完成pool-1-thread-1
小明锁已经释放
小强: 被锁定pool-1-thread-2
小强: 努力工作pool-1-thread-2
小强: 工作已经完成pool-1-thread-2
小强锁已经释放
小米: 被锁定pool-1-thread-3
小米: 努力工作pool-1-thread-3
小米: 工作已经完成pool-1-thread-3
小米锁已经释放
```



## ReadWriteLock

ReadWriteLock也是一个接口，在它里面只定义了两个方法：

```java
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading.
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing.
     */
    Lock writeLock();
}
```

一个用来获取读锁，一个用来获取写锁。也就是说，将对临界资源的读写操作分成两个锁来分配给线程，从而使得多个线程可以同时进行读操作。

### 子类锁---ReentrantReadWriteLock

转载地址：https://www.cnblogs.com/zaizhoumo/p/7782941.html 

#### 概述

ReentrantReadWriteLock是Lock的另一种实现方式，我们已经知道了ReentrantLock是一个排他锁，同一时间只允许一个线程访问，而ReentrantReadWriteLock允许多个读线程同时访问，**但不允许写线程和读线程、写线程和写线程同时访问。**相对于排他锁，提高了并发性。在实际应用中，大部分情况下对共享数据（如缓存）的访问都是读操作远多于写操作，这时ReentrantReadWriteLock能够提供比排他锁更好的并发性和吞吐量。

读写锁内部维护了两个锁，一个用于读操作，一个用于写操作。所有 ReadWriteLock实现都必须保证 writeLock操作的内存同步效果也要保持与相关 readLock的联系。也就是说，成功获取读锁的线程会看到写入锁之前版本所做的所有更新。

#### ReentrantReadWriteLock支持以下功能

1）支持公平和非公平的获取锁的方式；

2）支持可重入。读线程在获取了读锁后还可以获取读锁；写线程在获取了写锁之后既可以再次获取写锁又可以获取读锁；

3）还允许从写入锁降级为读取锁，其实现方式是：先获取写入锁，然后获取读取锁，最后释放写入锁。但是，从读取锁升级到写入锁是不允许的；

4）读取锁和写入锁都支持锁获取期间的中断；

5）Condition支持。仅写入锁提供了一个 Conditon 实现；读取锁不支持 Conditon ，readLock().newCondition() 会抛出 UnsupportedOperationException。 



#### 代码示例

模拟场景， 三个动漫， 只有浏览到是进击的巨人的动漫的时候，必须看完才能继续进行影片的浏览。 

```java
public class ReentrantReadWriteLockTest {

    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(10);
        ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
        WatchTV watch = new WatchTV(lock, "海绵宝宝");
        for (int i = 0; i < 10; i++) {
            service.execute(new Runnable() {
                @Override
                public void run() {
                    watch.watchTime();
                }
            });
        }
        service.shutdown();
    }

}

class WatchTV {

    private String tvName;

    private ReentrantReadWriteLock lock;

    private Lock writeLock;
    private Lock readLock;

    public WatchTV(String tvName) {
        this.tvName = tvName;
    }

    public WatchTV(ReentrantReadWriteLock lock, String tvName) {
        this.lock = lock;
        this.tvName = tvName;
        writeLock = lock.writeLock();
        readLock = lock.readLock();
    }

    public WatchTV(ReentrantReadWriteLock lock) {
        this.lock = lock;
    }

    public void watchTime() {
        findTv();
        System.out.println(Thread.currentThread().getName() + " 开始查看有什么好看的动漫");
        try {
            switch (this.tvName) {
            case "海绵宝宝":
                readLock.lock();
                System.out.println(Thread.currentThread().getName() + "-->找到: 海绵宝宝 不看，换台");
                findTv();
                readLock.unlock();
                break;
            case "死神":
                readLock.lock();
                System.out.println(Thread.currentThread().getName() + "-->找到: 死神  不看，换台");
                findTv();
                readLock.unlock();
                break;
            case "进击的巨人":
                writeLock.lock();
                System.out.println(Thread.currentThread().getName() + "-->找到: 进击的巨人--->不要换台，我要看看");
                TimeUnit.SECONDS.sleep(5);
                System.out.println(Thread.currentThread().getName() + "-->已经看完: 进击的巨人--->继续换台");
                findTv();
                writeLock.unlock();
                break;
            default:
                break;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    private void findTv() {
        List<String> names = Arrays.asList("海绵宝宝", "进击的巨人", "死神");
        int index = (int) (Math.random() * names.size());
        this.tvName = names.get(index);
    }
}
```

运行结果

```java
pool-1-thread-1 开始查看有什么好看的动漫
pool-1-thread-6 开始查看有什么好看的动漫
pool-1-thread-9 开始查看有什么好看的动漫
pool-1-thread-9-->找到: 死神  不看，换台
pool-1-thread-2 开始查看有什么好看的动漫
pool-1-thread-2-->找到: 海绵宝宝 不看，换台
pool-1-thread-8 开始查看有什么好看的动漫
pool-1-thread-8-->找到: 死神  不看，换台
pool-1-thread-7 开始查看有什么好看的动漫
pool-1-thread-7-->找到: 海绵宝宝 不看，换台
pool-1-thread-5 开始查看有什么好看的动漫
pool-1-thread-5-->找到: 死神  不看，换台
pool-1-thread-3 开始查看有什么好看的动漫
pool-1-thread-3-->找到: 海绵宝宝 不看，换台
pool-1-thread-4 开始查看有什么好看的动漫
pool-1-thread-10 开始查看有什么好看的动漫
pool-1-thread-6-->找到: 海绵宝宝 不看，换台
pool-1-thread-1-->找到: 海绵宝宝 不看，换台
pool-1-thread-4-->找到: 进击的巨人--->不要换台，我要看看
pool-1-thread-4-->已经看完: 进击的巨人--->继续换台
pool-1-thread-10-->找到: 进击的巨人--->不要换台，我要看看
pool-1-thread-10-->已经看完: 进击的巨人--->继续换台
```

如上可以看到， 10条线程，全部先到了方法watchTime()里面， 然后如果是读的操作，都是可以几条线程一起读的。 但是到了写的操作的时候，就必须让写的操作完成才可以。 

**注意这个读的锁并没有直接去读取数据， 只是为了判断在读的时候， 有没有线程对公共数据进行写的操作， 如果没有的话， 后面方法的读都是并发操作的。具体可以参考博客  《读写锁分离-设计模式》，这个并发读是不获取锁的。**

#### 注意

ReentrantReadWriteLock 实现了 ReadWriteLock 接口( 注意，**ReentrantReadWriteLock 并没有实现 Lock 接口** )，其包含两个很重要的方法：readLock() 和 writeLock() 分别用来获取读锁和写锁，**并且这两个锁实现了Lock接口**。





## StampedLock

转载地址： https://my.oschina.net/benhaile/blog/264383 

### 概述

StampedLock是Java8引入的一种新的所机制,简单的理解,可以认为它是读写锁的一个改进版本,读写锁虽然分离了读和写的功能,使得读与读之间可以完全并发,但是读和写之间依然是冲突的,读锁会完全阻塞写锁,它使用的依然是悲观的锁策略.如果有大量的读线程,他也有可能引起写线程的饥饿。

而StampedLock则提供了一种乐观的读策略,这种乐观策略的锁非常类似于无锁的操作,**使得乐观锁完全不会阻塞写线程**

### 比ReentrantReadWriteLock优势

如果读取执行情况很多，写入很少的情况下，使用 ReentrantReadWriteLock 可能会使写入线程遭遇饥饿（Starvation）问题，也就是写入线程吃吃无法竞争到锁定而一直处于等待状态。

1. `StampedLock` 提供了读锁和写锁相互转换的功能，使得该类支持更多的应用场景。

2. StampedLock的内部实现是基于CLH锁的,CLH锁是一种自旋锁,它保证没有饥饿的发生,并且可以保证FIFO(先进先出)的服务顺序.

3. 有一个**乐观读取的机制**。 

   



### StampedLock的三种状态

原文链接：https://blog.csdn.net/LightOfMiracle/article/details/73201442

**写入（Writing）**：writeLock方法获取独占式同步状态而阻塞当前线程，返回一个stamp，这个stamp能够在unlockWrite方法中使用从而释放锁。也提供了tryWriteLock。当锁被写模式所占有，读或者乐观的读操作都不能获取到同步状态。

**读取（Reading）**：readLock方法获取共享式同步状态而阻塞随后的写线程，返回一个stamp变量，能够在unlockRead方法中用于释放锁。同时也提供了tryReadLock方法。

**乐观读取（Optimistic Reading）**：提供了tryOptimisticRead方法返回一个非0的stamp，只有当前同步状态没有被写模式所占有是才能获取到。如果在获得stamp变量之后没有被写模式持有，方法validate将返回true。这种模式可以被看做一种弱版本的读锁，可以被一个写入者在任何时间打断。乐观读取模式仅用于短时间读取操作时经常能够降低竞争和提高吞吐量。同时使用的时候一般需要读取并存储到另外一个副本，以用做对比使用。



### 代码示例

```java
public class StampedLockTest {

    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(6);
        StampedLock lock = new StampedLock();
        Book book = new Book("小蝌蚪找妈妈", lock);

        for (int i = 0; i < 3; i++) {
            service.execute(new Runnable() {
                @Override
                public void run() {
                    book.update();
                }
            });
        }

        for (int i = 0; i < 3; i++) {
            service.execute(new Runnable() {
                @Override
                public void run() {
                    book.watch();
                }
            });
        }

        service.shutdown();

    }

}

class Book {

    private String name;
    private StampedLock lock;// 定义了StampedLock锁,

    public Book(String name, StampedLock lock) {
        this.name = name;
        this.lock = lock;
    }

    public void update() {
        long stamp = lock.writeLock();// 这里的含义和distanceFormOrigin方法中 s1.readLock()是类似的

        try {
            this.name = "大江东去";
            TimeUnit.SECONDS.sleep(2);
            System.out.println("写操作---->" + Thread.currentThread().getName() + " " + this.name +"  "+ new Date().getTime());
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            System.out.println("修改完成  " + new Date().getTime());
            lock.unlockWrite(stamp);
        }
    }

    public void watch() {
        // 试图尝试一次乐观读 返回一个类似于时间戳的邮戳整数stamp 这个stamp就可以作为这一个所获取的凭证
        long stamp = lock.tryOptimisticRead(); 
        // 判断这个stamp是否在读过程发生期间被修改过,如果stamp没有被修改过,责任无这次读取时有效的,因此就可以直接return了,反之,如果stamp是不可用的,则意味着在读取的过程中,可能被其他线程改写了数据,因此,有可能出现脏读,如果如果出现这种情况,我们可以像CAS操作那样在一个死循环中一直使用乐观锁,知道成功为止。
        if (!lock.validate(stamp)) {
            try {
                // stamp = lock.readLock();//也可以升级锁的级别,这里我们升级乐观锁的级别,将乐观锁变为悲观锁, 如果当前对象正在被修改,则读锁的申请可能导致线程挂起.
                TimeUnit.SECONDS.sleep(2);
                System.out.println("读操作---->" + Thread.currentThread().getName() + " " + this.name +"  "+ new Date().getTime());
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                // System.out.println("读取完成  " + new Date().getTime());
                // lock.unlockRead(stamp);// 退出临界区,释放读锁
            }
        } else {
            // watch();
        }
    }

}
```

运行结果

```java
写操作---->pool-1-thread-1 大江东去  1576641410875
读操作---->pool-1-thread-5 大江东去  1576641410875
读操作---->pool-1-thread-6 大江东去  1576641410875
读操作---->pool-1-thread-4 大江东去  1576641410875
修改完成  1576641410875
写操作---->pool-1-thread-3 大江东去  1576641412876
修改完成  1576641412876
写操作---->pool-1-thread-2 大江东去  1576641414881
修改完成  1576641414881
```

可以看到这里在写的时候， 我们的读的操作也是在进行中的。 并没有需要等他锁释放后在读取的。 

如果把上面的**将乐观锁变为悲观锁**

```java
public void watch() {

        long stamp = lock.tryOptimisticRead();  
        if (!lock.validate(stamp)) { 
            try {
                stamp = lock.readLock();
                TimeUnit.SECONDS.sleep(2);
                System.out.println("读操作---->" + Thread.currentThread().getName() + " " + this.name +"  "+ new Date().getTime());
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                System.out.println("读取完成  " + new Date().getTime());
                lock.unlockRead(stamp);// 退出临界区,释放读锁
            }
        } else {
            // watch();
        }
    }
```

运行结果如下：

```java
写操作---->pool-1-thread-1 大江东去  1576642637434
修改完成  1576642637434
写操作---->pool-1-thread-2 大江东去  1576642639436
修改完成  1576642639436
写操作---->pool-1-thread-3 大江东去  1576642641441
修改完成  1576642641441
读操作---->pool-1-thread-5 大江东去  1576642643446
读操作---->pool-1-thread-4 大江东去  1576642643446
读操作---->pool-1-thread-6 大江东去  1576642643446
读取完成  1576642643446
读取完成  1576642643446
读取完成  1576642643446
```

可以看到， 必须等写的锁释放完成后， 在可以进行读的操作。 



### 总结

在写的时候， writeLock的时候，照样可以进行读取的操作。 同时，**这个StampedLock完全可以代替ReentrantReadWriteLock这个类。** 



## Condition

转载地址： https://www.cnblogs.com/gemine/p/9039012.html

### 概述

在使用Lock之前，我们使用的最多的同步方式应该是synchronized关键字来实现同步方式了。配合Object的wait()、notify()系列方法可以实现等待/通知模式。Condition接口也提供了类似Object的监视器方法，与Lock配合可以实现等待/通知模式，但是这两者在使用方式以及功能特性上还是有差别的。Object和Condition接口的一些对比。摘自《Java并发编程的艺术》

###  Condition & wait & NotifyAll 对比

![Xnip2019-12-17_12-16-26](/img/Xnip2019-12-17_12-16-26.png)



### 常用方法

condition可以通俗的理解为条件队列。当一个线程在调用了await方法以后，直到线程等待的某个条件为真的时候才会被唤醒。这种方式为线程提供了更加简单的等待/通知模式。Condition必须要配合锁一起使用，因为对共享状态变量的访问发生在多线程环境下。一个Condition的实例必须与一个Lock绑定，因此Condition一般都是作为Lock的内部实现。

#### 1. await() 

造成当前线程在接到信号或被中断之前一直处于等待状态。

#### 2. await(long time, TimeUnit unit)

造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。

#### 3. awaitNanos(long nanosTimeout)

造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。返回值表示剩余时间，如果在nanosTimesout之前唤醒，那么返回值 = nanosTimeout - 消耗时间，如果返回值 <= 0 ,则可以认定它已经超时了。

#### 4. awaitUninterruptibly()

造成当前线程在接到信号之前一直处于等待状态。【注意：该方法对中断不敏感】。

#### 5. awaitUntil(Date deadline)

造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态。如果没有到指定时间就被通知，则返回true，否则表示到了指定时间，返回返回false。

#### 6. signal()

唤醒一个等待线程。该线程从等待方法返回前必须获得与Condition相关的锁。

#### 7. signal()All

唤醒所有等待线程。能够从等待方法返回的线程必须获得与Condition相关的锁。



### Condition接口原理简单解析

转载地址： https://www.cnblogs.com/gemine/p/9039012.html

#### 总结

调用await方法后，将当前线程加入Condition等待队列中。当前线程释放锁。否则别的线程就无法拿到锁而发生死锁。自旋(while)挂起，不断检测节点是否在同步队列中了，如果是则尝试获取锁，否则挂起。当线程被signal方法唤醒，被唤醒的线程将从await()方法中的while循环中退出来，然后调用acquireQueued()方法竞争同步状态。



### 生产者和消费者模式 

```java
public class BoundedQueue {

    private final LinkedList<Object> buffer; // 生产者容器
    private final int maxSize; // 容器最大值是多少
    private final Lock lock;
    private final Condition produceCondition;
    private final Condition consumerCondition;

    BoundedQueue(final int maxSize) {
        this.maxSize = maxSize;
        buffer = new LinkedList<Object>();
        lock = new ReentrantLock();
        produceCondition = lock.newCondition();
        consumerCondition = lock.newCondition();
    }

    /**
     * 生产者
     * 
     * @param obj
     * @throws InterruptedException
     */
    public void put(final Object obj) throws InterruptedException {
        lock.lock(); // 获取锁
        TimeUnit.SECONDS.sleep(1);
        System.out.println("生产方： " + Thread.currentThread().getName() + " 获取到锁 ");
        try {
            while (maxSize == buffer.size()) {
                System.out.println("生产已经满了.....");
                produceCondition.await(); // 满了，添加的线程进入等待状态
            }
            buffer.add(obj);
            System.out.println(obj);
            consumerCondition.signal(); // 通知
        } finally {
            lock.unlock();
        }
    }

    /**
     * 消费者
     * 
     * @return
     * @throws InterruptedException
     */
    public Object get() throws InterruptedException {
        lock.lock();
        Object obj;
        TimeUnit.SECONDS.sleep(1);
        System.out.println("消费方：" + Thread.currentThread().getName() + " 获取到锁 ");
        try {
            while (buffer.size() == 0) { // 队列中没有数据了 线程进入等待状态
                System.out.println("产品已经卖完了.....");
                consumerCondition.await();
            }
            obj = buffer.poll();
            System.out.println("卖掉产品: " + obj);
            produceCondition.signalAll(); // 通知
        } finally {
            lock.unlock();
        }
        return obj;
    }

    public static void main(final String[] args) {
        final BoundedQueue queue = new BoundedQueue(5);
        final ExecutorService service = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            final int index = i;
            service.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        queue.put("生成-" + index);
                    } catch (final InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }

        for (int i = 0; i < 5; i++) {
            service.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        queue.get();
                    } catch (final InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
        service.shutdown();
    }
}
```

运行结果

```java
生产方： pool-1-thread-1 获取到锁 
生成-0
生产方： pool-1-thread-2 获取到锁 
生成-1
生产方： pool-1-thread-3 获取到锁 
生成-2
生产方： pool-1-thread-4 获取到锁 
生成-3
生产方： pool-1-thread-5 获取到锁 
生成-4
消费方：pool-1-thread-6 获取到锁 
卖掉产品: 生成-0
消费方：pool-1-thread-7 获取到锁 
卖掉产品: 生成-1
消费方：pool-1-thread-8 获取到锁 
卖掉产品: 生成-2
消费方：pool-1-thread-9 获取到锁 
卖掉产品: 生成-3
消费方：pool-1-thread-10 获取到锁 
卖掉产品: 生成-4
```
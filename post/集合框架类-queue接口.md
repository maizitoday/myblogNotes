---
title:       "集合框架类-queue接口"
subtitle:    ""
description: ""
date:        2019-12-24
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["多线程和高并发", "java基础"]
categories:  ["Tech" ]
---

[TOC]

# Queue接口结构图

![20190321160846610](/img/20190321160846610.png)

# Queue接口

转载地址： https://blog.csdn.net/qq407388356/article/details/78319585 

转载地址： https://www.jianshu.com/p/7a86c56c632b

## 概述

队列(queue)是一种常用的数据结构，可以将队列看做是一种特殊的线性表，该结构遵循的先进先出原则。Java中，LinkedList实现了Queue接口,因为LinkedList进行插入、删除操作效率较高。

## 常用方法

```java
// 将元素追加到队列末尾,若添加成功则返回true。
boolean boolean offer(E e)
// 在尾部添加
boolean add(E)
 
// 从队首删除并返回该元素。  
E poll()
// 删除并返回头部
E remove()
  
 
// 返回队首元素，但是不删除  
E peek()
E element()
  
```

### add(E), offer(E) 区别

不同之处在于 add() 方法在添加失败（比如队列已满）时会报 一些运行时错误 错；而 offer() 方法即使在添加失败时也不会奔溃，只会返回 false。

**注意**：Queue 是个接口，它提供的 add, offer 方法初衷是希望子类能够禁止添加元素为 null，这样可以避免在查询时返回 null 究竟是正确还是错误。事实上大多数 Queue 的实现类的确响应了 Queue 接口的规定，比如 ArrayBlockingQueue，PriorityBlockingQueue 等等。但还是有一些实现类没有这样要求，比如 LinkedList。

### remove(), poll() 区别

当队列为空时 remove() 方法会报 NoSuchElementException 错; 而 poll() 不会奔溃，只会返回 null。

### element(), peek() 区别

当队列为空时 element() 抛出异常；peek() 不会奔溃，只会返回 null。





## 常用子类

### LinkedList

#### 概述

双向队列(Deque)，是Queue的一个子接口，双向队列是指该队列两端的元素既能入队(offer)也能出队(poll)，如果将Deque限制为只能从一端入队和出队，则可实现栈的数据结构。对于栈而言，有入栈(push)和出栈(pop)，遵循先进后出原则。

#### 常用方法

```java
// 将给定元素”压入”栈中。存入的元素会在栈首。即:栈的第一个元素
void push(E e)

// 将栈首元素删除并返回。
E pop()
```

#### 注意

**ArrayDeque`和`LinkeList`实现了`Deque接口**



### PriorityQueue(优先队列)

转载地址：https://www.jianshu.com/p/c577796e537a

#### 概述

PriorityQueue 是基于优先堆的一个无界队列，这个优先队列中的元素可以默认自然排序或者通过提供的Comparator 在队列实例化的时排序。

**PriorityQueue 不允许空值**，而且不支持 non-comparable（不可比较）的对象，比如用户自定义的类。优先队列要求使用 Java Comparable 和 Comparator 接口给对象排序，并且在排序时会按照优先级处理其中的元素。

**PriorityQueue 是非线程安全的**，所以 Java 提供了 PriorityBlockingQueue（实现 BlockingQueue接口）用于Java 多线程环境。

#### 示例代码

转载地址：https://blog.csdn.net/cyp331203/article/details/25310733

```java
PriorityQueue pq = new PriorityQueue();
pq.offer(6);
pq.offer(-3);
pq.offer(0);
pq.offer(9);
System.out.println(pq);

运行结果
[-3, 6, 0, 9]
```

不是说是按从小到大来排序的吗？怎么没排序？

**关键在于PriorityQueue的排序不是普通的排序，而是堆排序**

原因是堆排序只会保证第一个元素也就是根节点的元素是当前优先队列里最小（或者最大）的元素，而且每一次变化之后，比如offer()或者poll()之后，都会进行堆重排，所以如果想要按从小到大的顺序取出元素，那么需要用一个for循环

```java
import java.util.PriorityQueue;
public class PriorityQueueTest3 
{
	public static void main(String[] args) 
	{
		PriorityQueue pq = new PriorityQueue();
		pq.offer(6);
		pq.offer(-3);
		pq.offer(0);
		pq.offer(9);
    //这里之所以取出.size()大小，因为每一次poll()之后size大小都会变化，所以不能作为for循环的判断条件
		int len = pq.size();
		for(int i=0;i<len;i++){
			System.out.print(pq.poll()+" ");
		}
		System.out.println();
	}
}
```

#### 自定义排序

```java
public class CollectionTest {

    public static void main(String[] args) {

        Person pp1 = new Person("小米", 10);
        Person pp2 = new Person("小花", 2);
        Person pp3 = new Person("小猫", 1);

        PriorityQueue<Person> queue = new PriorityQueue<Person>();
        queue.offer(pp1);
        queue.offer(pp2);
        queue.offer(pp3);

        queue.forEach(System.out::println);  // 1 

        System.out.println( queue.poll());  //  2 
        System.out.println( queue.poll());
        System.out.println( queue.poll());
      
    }
}

class Person implements Comparable<Person> {

    public String name;
    public int age;

    public Person(String name, int age) {
        this.age = age;
        this.name = name;
    }

    @Override
    public String toString() {
        return this.name + ":" + this.age;
    }

    @Override
    public int compareTo(Person o) {

        return this.age - o.age;
    }

}

```

运行代码1结果

```java
小猫:1
小米:10
小花:2
```

运行代码2结果

```java
小猫:1
小花:2
小米:10
```



## Deque接口(双端队列)

转载地址：https://www.jianshu.com/p/e3ce916f8db4

### 概述

`Deque`表示双端队列。双端队列是在两端都可以进行插入和删除的队列。`Deque`是一个比`Stack`和`Queue`功能更强大的接口，它同时实现了栈和队列的功能。**ArrayDeque`和`LinkeList`实现了`Deque接口**。

**注意**：`Deque`既可以用作后进先出的栈，也可以用作先进先出的队列。 **Deque 扩展了 Queue**，有队列的所有方法，还可以看做栈

### 常用方法

![Xnip2019-12-24_15-54-10](/img/Xnip2019-12-24_15-54-10.png)

#### 插入

`addFirst`和`offerFirst`在`Deque`实例头部插入元素。
 `addLast`和`offerLast`在`Deque`实例尾部插入元素。
 当`Deque`实现类为有限容量时，优先使用`offerFirst`和`offerLast`，因为`addFirst`在队列满的时候可能会插入失败而抛出异常。

#### 删除

`removeFirst`和`pollFirst`从`Deque`实例头部移除元素。
 `removeLast`和`pollLast`从`Deque`实例尾部移除元素。
 当`Deque`为空时，`pollFirst`和`pollLast`将会返回`null`，而`removeFirst`和`removeLast`将会抛出异常。

#### 检索

`getFirst`和`peekFirst`获取`Deque`实例的第一个元素，但是不会将元素从`Deque`实例中删除。类似地，`getLast`和`peekLast`获取最后一个元素。当`Deque`为空时，`getFirst`和`getLast`将会抛出异常，而`peekFirst`和`peekLast`将会返回`null`。

#### 其他

除了基本的插入、删除和检索方法外，还有两个预定义的方法：`removeFirstOccurence`和`removeLastOccurence`。这两个方法见名知意。返回`true`的时候表示元素存在于队列，并且已经被删除。返回`false`时表示元素不存在于队列中，并且队列没有改变。

 

### 子类---ArrayDeque

##### 概述

ArrayDeque 是一个用数组实现的双端队列 Deque，为满足可以同时在数组两端插入或删除元素的需求，该数组还必须是循环的，**即循环数组（circular array），**也就是说数组的任何一点都可能被看作起点或者终点，**ArrayDeque 是非线程安全的，**当多个线程同时使用的时候需要手动同步，此外**该容器不允许放 null 元素，同时与 ArrayList 和 LinkedList 不同的是没有索引位置的概念，不能根据索引位置进行操作。**

 

## BlockingQueue接口(阻塞队列)

转载地址：https://blog.csdn.net/wuzhiwei549/article/details/79869699

### 概述

阻塞队列 (BlockingQueue)是Java util.concurrent包下重要的数据结构，BlockingQueue提供了线程安全的队列访问方式：当阻塞队列进行插入数据时，如果队列已满，线程将会阻塞等待直到队列非满；从阻塞队列取数据时，如果队列已空，线程将会阻塞等待直到队列非空。阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。并发包下很多高级同步类的实现都是基于BlockingQueue实现的。
 

### 核心方法

```java
offer(E e): 将给定的元素设置到队列中，如果设置成功返回true, 否则返回false. e的值不能为空，否则抛出空指针异常。
offer(E e, long timeout, TimeUnit unit): 将给定元素在给定的时间内设置到队列中，如果设置成功返回true, 否则返回false.
add(E e): 将给定元素设置到队列中，如果设置成功返回true, 否则抛出异常。如果是往限定了长度的队列中设置值，推荐使用offer()方法。

put(E e): 将元素设置到队列中，如果队列中没有多余的空间，该方法会一直阻塞，直到队列中有多余的空间。
take(): 从队列中获取值，如果队列中没有值，线程会一直阻塞，直到队列中有值，并且该方法取得了该值。
poll(long timeout, TimeUnit unit): 获取并移除此队列的头元素，可以在指定的等待时间前等待可用的元素，timeout表明放弃之前要等待的时间长度，用 unit 的时间单位表示，如果在元素可用前超过了指定的等待时间，则返回null，当等待时可以被中断
remainingCapacity()：获取队列中剩余的空间。
remove(Object o): 从队列中移除指定的值。
contains(Object o): 判断队列中是否拥有该值。
drainTo(Collection c): 将队列中值，全部移除，并发设置到给定的集合中。
```

**注意：** put(E e) 和 take() 方法。



### 总结

BlockingQueue不光实现了一个完整队列所具有的基本功能，同时在多线程环境下，他还自动管理了多线间的自动等待于唤醒功能，从而使得程序员可以忽略这些细节，关注更高级的功能。 



### 常用子类

转载地址： https://www.cnblogs.com/jackyuj/archive/2010/11/24/1886553.html 



#### ArrayBlockingQueue

##### 概述

ArrayBlockingQueue 是一个有界的阻塞队列，其内部实现是将对象放到一个数组里。有界也就意味着，它不能够存储无限多数量的元素。它有一个同一时间能够存储元素数量的上限。你可以在对其初始化的时候设定这个上限，但之后就无法对这个上限进行修改了(译者注：因为它是基于数组实现的，也就具有数组的特性：一旦初始化，大小就无法修改)。



#### DelayQueue

##### 概述

DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue是一个没有大小限制的队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞。

##### 使用场景

DelayQueue使用场景较少，但都相当巧妙，常见的例子比如使用一个DelayQueue来管理一个超时未响应的连接队列



#### LinkedBlockingQueue

##### 概述

基于链表的阻塞队列，同ArrayListBlockingQueue类似，其内部也维持着一个数据缓冲队列（该队列由一个链表构成），当生产者往队列中放入一个数据时，队列会从生产者手中获取数据，并缓存在队列内部，而生产者立即返回；只有当队列缓冲区达到最大值缓存容量时（LinkedBlockingQueue可以通过构造函数指定该值），才会阻塞生产者队列，直到消费者从队列中消费掉一份数据，生产者线程会被唤醒，反之对于消费者这端的处理也基于同样的原理。而LinkedBlockingQueue之所以能够高效的处理并发数据，还因为其对于生产者端和消费者端分别采用了独立的锁来控制数据同步，这也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能。
作为开发者，我们需要注意的是，如果构造一个LinkedBlockingQueue对象，而没有指定其容量大小，LinkedBlockingQueue会默认一个类似无限大小的容量（Integer.MAX_VALUE），这样的话，如果生产者的速度一旦大于消费者的速度，也许还没有等到队列满阻塞产生，系统内存就有可能已被消耗殆尽了。



#### PriorityBlockingQueue

##### 概述

PriorityBlockingQueue 是一个无界的并发队列。它使用了和类 java.util.PriorityQueue 一样的排序规则。你无法向这个队列中插入 null 值。所有插入到 PriorityBlockingQueue 的元素必须实现 java.lang.Comparable 接口。因此该队列中元素的排序就取决于你自己的 Comparable 实现。

基于优先级的阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定），但需要注意的是PriorityBlockingQueue并不会阻塞数据生产者，而只会在没有可消费的数据时，阻塞数据的消费者。因此使用的时候要特别注意，生产者生产数据的速度绝对不能快于消费者消费数据的速度，否则时间一长，会最终耗尽所有的可用堆内存空间。在实现PriorityBlockingQueue时，内部控制线程同步的锁采用的是公平锁。



#### SynchronousQueue

##### 概述

SynchronousQueue 是一个特殊的队列，它的内部同时只能够容纳单个元素。如果该队列已有一元素的话，试图向队列中插入一个新元素的线程将会阻塞，直到另一个线程将该元素从队列中抽走。同样，如果该队列为空，试图向队列中抽取一个元素的线程将会阻塞，直到另一个线程向队列中插入了一条新的元素。据此，把这个类称作一个队列显然是夸大其词了。它更多像是一个汇合点

##### 使用场景

声明一个SynchronousQueue有两种不同的方式，它们之间有着不太一样的行为。公平模式和非公平模式的区别:
**公平模式**：SynchronousQueue会采用公平锁，并配合一个FIFO队列来阻塞多余的生产者和消费者，从而体系整体的公平策略；
**非公平模式**（SynchronousQueue默认）：SynchronousQueue采用非公平锁，同时配合一个LIFO队列来管理多余的生产者和消费者，而后一种模式，如果生产者和消费者的处理速度有差距，则很容易出现饥渴的情况，即可能有某些生产者或者是消费者的数据永远都得不到处理。



#### LinkedBlockingDeque

##### 概述

1. 该队列是基于双向链表的有界阻塞队列。
2. 构造函数的可选参数是容量的界限，用来阻止容量过度扩展的一种方法。也就是说，虽然是双向链表，但也可以指定其容量大小。容量（如果未指定）等于Integer.MAX_VALUE。若节点数量没有到达容量的界限，链接节点在每次插入时都会动态创建。
3. 大多数操作在恒定时间运行，异常包括remove，removeFirstOccurrence，removeLastOccurrence，contains，iterator.remove()和批量操作，所有这些都是以线性时间运行的。
4. 可以指定元素的插入位置：头部还是尾部。
    

#### LinkedTransferQueue

转载链接：https://www.jianshu.com/p/ae6977886cec

##### 概述

相比较普通的阻塞队列，增加了这么几个方法。

```java
public interface TransferQueue<E> extends BlockingQueue<E> {
    // 如果可能，立即将元素转移给等待的消费者。 
    // 更确切地说，如果存在消费者已经等待接收它（在 take 或 timed poll（long，TimeUnit）poll）中，则立即传送指定的元素，否则返回 false。
    boolean tryTransfer(E e);

    // 将元素转移给消费者，如果需要的话等待。 
    // 更准确地说，如果存在一个消费者已经等待接收它（在 take 或timed poll（long，TimeUnit）poll）中，则立即传送指定的元素，否则等待直到元素由消费者接收。
    void transfer(E e) throws InterruptedException;

    // 上面方法的基础上设置超时时间
    boolean tryTransfer(E e, long timeout, TimeUnit unit) throws InterruptedException;

    // 如果至少有一位消费者在等待，则返回 true
    boolean hasWaitingConsumer();

    // 返回等待消费者人数的估计值
    int getWaitingConsumerCount();
}
```

##### 整个过程如下图

![Xnip2019-12-24_17-38-05](/img/Xnip2019-12-24_17-38-05.png)

##### 总结

`LinkedTransferQueue`是 `SynchronousQueue` 和 `LinkedBlockingQueue` 的合体，性能比 `LinkedBlockingQueue` 更高（没有锁操作），比 `SynchronousQueue`能存储更多的元素。

当 `put` 时，如果有等待的线程，就直接将元素 “交给” 等待者， 否则直接进入队列。

`put`和 `transfer` 方法的区别是，put 是立即返回的， transfer 是阻塞等待消费者拿到数据才返回。`transfer`方法和 `SynchronousQueue`的 put 方法类似。



## ConcurrentLinkedQueue

### 概述

在并发编程中我们有时候需要使用线程安全的队列。如果我们要实现一个线程安全的队列有两种实现方式一种是使用阻塞算法，另一种是使用非阻塞算法。使用阻塞算法的队列可以用一个锁（入队和出队用同一把锁）或两个锁（入队和出队用不同的锁）等方式来实现，而非阻塞的实现方式则可以使用循环CAS的方式来实现，下面我们一起来研究下Doug Lea是如何使用非阻塞的方式来实现线程安全队列ConcurrentLinkedQueue的。

ConcurrentLinkedQueue是一个基于链接节点的无界线程安全队列，它采用先进先出的规则对节点进行排序，当我们添加一个元素的时候，它会添加到队列的尾部，当我们获取一个元素时，它会返回队列头部的元素。它采用了“wait－free”算法来实现。

### 线程安全的队列哪几种方法

原文链接：https://blog.csdn.net/lifuxiangcaohui/article/details/8051144

链接：https://www.jianshu.com/p/231caf90f30b

**第一种：**使用synchronized同步队列，就像Vector或者Collections.synchronizedList/Collection那样。
显然这不是一个好的并发队列，这会导致吞吐量急剧下降。

**第二种：**使用Lock。一种好的实现方式是使用ReentrantReadWriteLock来代替ReentrantLock提高读取的吞吐量。
 但是显然 ReentrantReadWriteLock的实现更为复杂，而且更容易导致出现问题，另外也不是一种通用的实现方式，因为 ReentrantReadWriteLock适合哪种读取量远远大于写入量的场合。当然了ReentrantLock是一种很好的实现，结合 Condition能够很方便的实现阻塞功能，

**第三种：**使用CAS操作。尽管Lock的实现也用到了CAS操作，但是毕竟是间接操作，而且会导致线程挂起。
 一个好的并发队列就是采用某种非阻塞算法来取得最大的吞吐量。ConcurrentLinkedQueue采用的就是第三种策略。

### ConcurrentLinkedQueue的结构图

![Xnip2019-12-24_18-09-46](/img/Xnip2019-12-24_18-09-46.png)

从内图可以了解ConcurrentLinkedQueue一个大概，ConcurrentLinkedQueue内部持有2个节点：head头结点，负责出列， tail尾节点，负责入列。而元素节点Node，使用item存储入列元素，next指向下一个元素节点。

```java
    private static class Node<E> {
        volatile E item;
        volatile Node<E> next;
        //....
    }

```

```java
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
        implements Queue<E>, java.io.Serializable {
        private transient volatile Node<E> head;
        private transient volatile Node<E> tail;  
        //....
}

```

### 使用特点

1. 不允许null入列
2. 在入队的最后一个元素的next为null
3. 队列中所有未删除的节点的item都不能为null且都能从head节点遍历到
4. 删除节点是将item设置为null, 队列迭代时跳过item为null节点
5. head节点跟tail不一定指向头节点或尾节点，可能存在滞后性

# 常见问题

## ArrayDeque 与 LinkedList 的适用场景

转载地址：https://www.jianshu.com/p/1e75bf192f9f

ArrayDeque 和 LinkedList 都实现了 Deque 接口，如果只需要 Deque 接口且从两端进行操作则一般来说 ArrayDeque 效率更高一些，如果同时需要根据索引位置进行操作或经常需要在中间进行插入和删除操作则应该优先选 LinkedList 效率高些，因为 **ArrayDeque 实现是循环数组结构（即一个动态扩容数组，默认容量 16，然后通过一个 head 和 tail 索引记录首尾相连），而 LinkedList 是基于双向链表结构的。**



## LinkedBlockingDeque和ArrayBlockQueue的对比

原文链接：https://blog.csdn.net/zg_hover/article/details/77984209

1. ArrayBlockQueue的队列大小是固定的，LinkedBlockingDeque的队列大小可以固定，也可以不固定。
2. LinkedBlockingDeque中添加和删除节点是动态的，有内存的申请和释放，性能可能有损耗。
   而ArrayBlockQueue只是把某个位置的数据置为空，队列总的容量大小不会变化，变化的队列中的元素个数。
3. 在性能要求较高的环境下，建议使用ArrayBlockQueue。
4. LinkedBlockingDeque是一个双向队列，尾部或头部都可以插入或获取接节点。



# 总结

ArrayBlockingQueue和LinkedBlockingQueue是两个最普通也是最常用的阻塞队列，一般情况下，在处理多线程间的生产者消费者问题，**使用这两个类足以。**

队列的使用率很高，多数生产消费模型的首选数据结构就是队列(先进先出)。Java提供的线程安全的Queue可以分为阻塞队列和非阻塞队列，其中阻塞队列的典型例子是BlockingQueue，非阻塞队列的典型例子是ConcurrentLinkedQueue，在实际应用中要根据实际需要选用阻塞队列或者非阻塞队列。



# 生产者和消费者

转载地址：https://blog.csdn.net/z69183787/article/details/81064982 

```java
public class Test01ConcurrentLinkedQueue {
    public static void main(String[] args) throws InterruptedException {
        int peopleNum = 10;//吃饭人数
        int tableNum = 10;//饭桌数量

        ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();
        CountDownLatch count = new CountDownLatch(tableNum);//计数器

        //将吃饭人数放入队列（吃饭的人进行排队）
        for(int i=1;i<=peopleNum;i++){
            queue.offer("消费者_" + i);
        }
        //执行10个线程从队列取出元素（10个桌子开始供饭）
        System.out.println("-----------------------------------开饭了-----------------------------------");
        long start = System.currentTimeMillis();
        ExecutorService executorService = Executors.newFixedThreadPool(tableNum);
        for(int i=0;i<tableNum;i++) {
            executorService.submit(new Dinner("00" + (i+1), queue, count));
        }
        //计数器等待，知道队列为空（所有人吃完）
        count.await();
        long time = System.currentTimeMillis() - start;
        System.out.println("-----------------------------------所有人已经吃完-----------------------------------");
        System.out.println("共耗时：" + time);
        //停止线程池
        executorService.shutdown();
    }

    private static class Dinner implements Runnable{
        private String name;
        private ConcurrentLinkedQueue<String> queue;
        private CountDownLatch count;

        public Dinner(String name, ConcurrentLinkedQueue<String> queue, CountDownLatch count) {
            this.name = name;
            this.queue = queue;
            this.count = count;
        }

        @Override
        public void run() {
            //while (queue.size() > 0){
            while (!queue.isEmpty()){
                //从队列取出一个元素 排队的人少一个
                System.out.println("【" +queue.poll() + "】----已吃完...， 饭桌编号：" + name);
            }
            count.countDown();//计数器-1
        }
    }
}
```

运行结果

```java
-----------------------------------开饭了-----------------------------------
【消费者_2】----已吃完...， 饭桌编号：002
【消费者_1】----已吃完...， 饭桌编号：001
【消费者_4】----已吃完...， 饭桌编号：003
【消费者_6】----已吃完...， 饭桌编号：003
【消费者_7】----已吃完...， 饭桌编号：004
【消费者_9】----已吃完...， 饭桌编号：004
【消费者_10】----已吃完...， 饭桌编号：004
【消费者_3】----已吃完...， 饭桌编号：002
【消费者_8】----已吃完...， 饭桌编号：003
【消费者_5】----已吃完...， 饭桌编号：001
-----------------------------------所有人已经吃完-----------------------------------
共耗时：6
```
---
title:       "集合框架类-Collection，Map接口"
subtitle:    ""
description: ""
date:        2019-12-18
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["多线程和高并发", "java基础"]
categories:  ["Tech" ]
---

[TOC]



**转载地址：https://blog.csdn.net/qq407388356/article/details/78319585**

# 集合框架

## 结构图

![20190321160846610](/img//20190321160846610.png)

## 概述

集合包最常用的有Collection和Map两个接口的实现类，Colleciton用于存放多个单对象，Map用于存放Key-Value形式的键值对。

Collection中最常用的又分为两种类型的接口：List和Set，两者最明显的差别为List支持放入重复的元素，而Set不支持。

## 常见集合框架介绍

![Xnip2019-12-19_13-53-46](/img//Xnip2019-12-19_13-53-46.png)

**说明：Queue这一块后续单独一篇文章。** 

# Collection 接口

## 概述

Collection 是最基本的集合接口，一个 Collection 代表一组 Object，即 Collection 的元素, Java不提供直接继承自Collection的类，只提供继承于的子接口(如List和set)。

## 子类接口

### List接口

#### 概述

List接口是一个有序的Collection，使用此接口能够精确的控制每个元素插入的位置，能够通过索引(元素在List中位置，类似于数组的小标)来访问List中的元素，而且允许有相同的元素。

#### 实现类

##### ArrayList

###### 概述

ArrayList基于数组方式实现，默认构造器通过调用ArrayList(int)来完成创建，传入的值为10，实例化了一个Object数组，并将此数组赋给了当前实例的elementData属性，此Object数组的大小即为传入的initialCapacity，因此调用空构造器的情况下会创建一个大小为10的Object数组。

###### 常用方法

**1.  插入对象：add(E)**

基于已有元素数量加1作为名叫minCapacity的变量，比较此值和Object数组的大小，若大于数组值，那么先将当前Object数组值赋给一个数组对象，接着产生一个新的数组容量值。此值的计算方法为当前数组值*1.5+1，如得出的容量值仍然小于minCapacity，那么就以minCapacity作为新的容量值，调用Arrays.copyOf来生成新的数组对象。

还提供了add(int,E)这样的方法将元素直接插入指定的int位置上，将目前index及其后的数据都往后挪一位，然后才能将指定的index位置的赋值为传入的对象，这种方式要多付出一次复制数组的代价。还提供了addAll。

**2.  删除对象：remove(E)**

这里调用了faseRemove方法将index后的**对象往前复制一位**，并将数组中的最后一个元素的值设置为null，即释放了对此对象的引用。还提供了remove(int)方法来删除指定位置的对象，remove(int)的实现比remove(E)多了一个数组范围的检测，但少了对象位置的查找，因此性能会更好。

**3. 获取单个对象：get(int)**

**4. 遍历对象：iterator()**

**5. 判断对象是否存在：contains(E)**

###### **总结**

1. ArrayList基于数组方式实现，无容量的限制；
2. ArrayList在执行插入元素时可能要扩容，在删除元素时并不会减小数组的容量（如希望相应的缩小数组容量，可以调用ArrayList的trimToSize()），**在查找元素时要遍历数组，对于非null的元素采取equals的方式寻找；**
3. ArrayList是非线程安全的。可以使用CopyOnWriteArrayList保证线程安全。



##### LinkedList

###### 概述

LinkedList基于双向链表机制，所谓双向链表机制，就是集合中的每个元素都知道其前一个元素及其后一个元素的位置。在LinkedList中，以一个内部的Entry类来代表集合中的元素，元素的值赋给element属性，Entry中的next属性指向元素的后一个元素，Entry中的previous属性指向元素的前一个元素，基于这样的机制可以快速实现集合中元素的移动。

###### 总结

1. LinkedList基于双向链表机制实现；
2. LinkedList在插入元素时，须创建一个新的Entry对象，并切换相应元素的前后元素的引用；在查找元素时，须遍历链表；在删除元素时，要遍历链表，找到要删除的元素，然后从链表上将此元素删除即可，此时原有的前后元素改变引用连在一起；
3. LinkedList是非线程安全的。 
4. 他也实现了Queue队列接口，他也是一个队列。 



##### Vector

###### 概述

Vector类实现了一个动态数组。和ArrayList和相似，但是两者是不同的：

1. Vector是同步访问的。

2. Vector包含了许多传统的方法，这些方法不属于集合框架。

其add、remove、get(int)方法都加了synchronized关键字，默认创建一个大小为10的Object数组，并将capacityIncrement设置为0。容量扩充策略：如果capacityIncrement大于0，则将Object数组的大小扩大为现有size加上capacityIncrement的值；如果capacity等于或小于0，则将Object数组的大小扩大为现有size的两倍，这种容量的控制策略比ArrayList更为可控。

###### 总结

**Vector是基于Synchronized实现的线程安全的ArrayList，**但在插入元素时容量扩充的机制和ArrayList稍有不同，并可通过传入capacityIncrement来控制容量的扩充。

###### 子类--Stack

栈是Vector的一个子类，**它实现了一个标准的后进先出的栈**。堆栈只定义了默认构造函数，用来创建一个空栈。Stack继承于Vector，在其基础上实现了Stack所要求的后进先出(LIFO)的弹出与压入操作，其提供了push、pop、peek三个主要的方法。

**注意：线程是安全的。**



##### CopyOnWriteArrayList

转载链接：https://blog.csdn.net/zongyeqing/article/details/81807681  

转载地址：https://www.xttblog.com/?p=4006

###### 概述

为了将读取的性能发挥到极致，jdk中提供了CopyOnWriteArrayList类。对它来说，读取是完全不用加锁的，写入也不会阻塞读取操作。只有写入与写入之间需要进行同步等待。那么它是如何做到的呢？从这个类的名字中可以看出，CopyOnWrite就是在写入操作时，进行一次自我复制。也就是对原有的数据进行一次复制，将修改的内容写入副本中。写完之后，再将修改完的副本替换原来的数据。这样就可以保证写操作不影响读了。

###### 几个要点

1. 实现了List接口
2. 内部持有一个ReentrantLock lock = new ReentrantLock();
3. 底层是用volatile transient声明的数组 array
4. 读写分离，写时复制出一个新的数组，完成插入、修改或者移除操作后将新数组赋值给array。

###### 缺点

1. 由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致 young gc 或者 full gc。
2. 不能用于实时读的场景，像拷贝数组、新增元素都需要时间，所以调用一个 set 操作后，读取到数据可能还是旧的，虽然CopyOnWriteArrayList 能做到最终一致性,但是还是没法满足实时性要求
3. 由于实际使用中可能没法保证 CopyOnWriteArrayList 到底要放置多少数据，万一数据稍微有点多，每次 add/set 都要重新复制数组，这个代价实在太高昂了。在高性能的互联网应用中，这种操作分分钟引起故障。

###### 设计思想

1. 读写分离，读和写分开
2. 最终一致性
3. 使用另外开辟空间的思路，来解决并发冲突

 

### Set接口

#### 概述

Set 具有与 Collection 完全一样的接口，只是行为上不同，Set 不保存重复的元素。

1. Set不允许包含相同的元素，如果试图把两个相同元素加入同一个集合中，add方法返回false。
2. Set判断两个对象相同不是使用==运算符，而是根据equals方法。也就是说，只要两个对象用equals方法比较返回true，Set就不会接受这两个对象。

**因为Object的equal方法默认是两个对象的引用的比较，意思就是指向同一内存,地址则相等，否则不相等；如果你现在需要利用对象里面的值来判断是否相等，则重载equal方法。先检索hashcode值，不等的情况下才会去比较equals方法的。**



#### 实现类

##### HashSet

###### 概述

默认构造创建一个HashMap对象，HashSet是基于HashMap实现的。

###### 常用方法

```java
// 调用HashMap的put方法来完成此操作，将需要增加的元素作为Map中的key，value则传入一个之前已创建的Object对象。
add(E)：  
```

```java
// 调用HashMap的remove(E)方法完成此操作。
remove(E)： 
```

```java
// HashMap的containsKey
contains(E)：
```

```java
// 调用HashMap的keySet的iterator方法。
iterator() ：   
```

###### 总结

1. HashSet基于HashMap实现，无容量限制；
2. **HashSet是非线程安全的**.



###### 子类--LinkedHashSet

LinkedHashSet集合也是根据元素**hashCode值来决定元素存储位置**，但它同时使用链表维护元素的次序，这样使得元素看起来是以插入的顺序保存的。也就是说，当遍历LinkedHashSet集合里元素时，HashSet将会按元素的添加顺序来访问集合里的元素。

LinkedHashSet需要维护元素的插入顺序，因此性能略低于HashSet的性能，但在迭代访问Set里的全部元素时将有很好的性能，因为它以链表来维护内部顺序。



##### TreeSet 

###### 概述

TreeSet和HashSet的主要不同在于TreeSet对于排序的支持，TreeSet基于TreeMap实现。**依托于Compared接口的处理。** 



##### ConcurrentSkipListSet

###### 概述

ConcurrentSkipListSet是**线程安全**的有序的集合，适用于高并发的场景。

ConcurrentSkipListSet和[TreeSet](http://www.cnblogs.com/skywang12345/p/3311268.html)，**它们虽然都是有序的集合**。但是，第一，它们的线程安全机制不同，TreeSet是非线程安全的，而ConcurrentSkipListSet是线程安全的。第二，ConcurrentSkipListSet是通过[ConcurrentSkipListMap](http://www.cnblogs.com/skywang12345/p/3498556.html)实现的，而TreeSet是通过TreeMap实现的。

###### 源码

```java
 public TreeSet(Comparator<? super E> comparator) {
        this(new TreeMap<>(comparator));
    }
```



##### CopyOnWriteArraySet

转载地址： https://www.cnblogs.com/xiaolovewei/p/9142046.html 

###### 概述

它是**线程安全**的无序的集合，可以将它理解成线程安全的[HashSet](http://www.cnblogs.com/skywang12345/p/3311252.html)。CopyOnWriteArraySet包含CopyOnWriteArrayList对象，它是通过CopyOnWriteArrayList实现的。而CopyOnWriteArrayList本质是个动态数组队列，

所以CopyOnWriteArraySet相当于通过通过动态数组实现的“集合”！ CopyOnWriteArrayList中允许有重复的元素；但是，**CopyOnWriteArraySet是一个集合，所以它不能有重复集合**。因此，CopyOnWriteArrayList额外提供了addIfAbsent()和addAllAbsent()这两个添加元素的API，通过这些API来添加元素时，只有当元素不存在时才执行添加操作！



###### 具有以下特性

1. 它最适合于具有以下特征的应用程序：Set 大小通常保持很小，只读操作远多于可变操作，需要在遍历期间防止线程间的冲突。
2. 它是线程安全的。
3. 因为通常需要复制整个基础数组，所以可变操作（add()、set() 和 remove() 等等）的开销很大。
4. 迭代器支持hasNext(), next()等不可变操作，但不支持可变 remove()等 操作。
5. 使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不变的数组快照。



##### EnumSet

转载地址： https://www.cnblogs.com/treeshu/p/11013511.html

###### 概述

EnumSet这是一个用来操作Enum的集合，是一个抽象类，它有两个继承类：JumboEnumSet和RegularEnumSet。在使用的时候，需要确定枚举类型。它的特点也是速度非常快，为什么速度很快呢？因为每次add的时候，每个枚举值只占一个长整型的一位。

###### 常用方法

![Xnip2019-12-21_15-58-00](/img//Xnip2019-12-21_15-58-00.png)

# Map接口

## 概述

将唯一的键映射到值。Map.Entry，描述在一个Map中的一个元素（键/值对）。**是一个Map的内部类**。

## 结构图

![p](/img//p.jpg)



## 实现类

### HashMap

#### 概述

基于数组+链表的结合体(链表散列)实现，当链表长度大于8时，则转换为红黑树实现，减少查找时间复杂度，将key-value看成一个整体，存放于Entity[]数组，put的时候根据key hash后的hashcode和数组length-1按位与的结果值判断放在数组的哪个位置，如果该数组位置上若已经存放其他元素，则在这个位置上的元素以链表的形式存放。如果该位置上没有元素则直接存放。

当系统决定存储HashMap中的key-value对时，完全没有考虑Entry中的value，仅仅只是根据key来计算并决定每个Entry的存储位置。我们完全可以把Map集合中的value当成key的附属，当系统决定了key的存储位置之后，value随之保存在那里即可。get取值也是根据key的hashCode确定在数组的位置，在根据key的equals确定在链表处的位置。

当HashMap中的元素越来越多的时候，hash冲突的几率也就越来越高，因为数组的长度是固定的。所以为了提高查询的效率，就要对HashMap的数组进行扩容，而在HashMap数组扩容之后，最消耗性能的点就出现了：原数组中的数据必须重新计算其在新数组中的位置，并放进去，这就是resize。那么HashMap什么时候进行扩容呢？当HashMap中的元素个数超过数组大小*loadFactor时，就会进行数组扩容，loadFactor的默认值为0.75，这是一个折中的取值。



#### Map集合的特点

1. Map集合是一个**双列集合**，一个元素包含两值（一个key，一个value）

2. Map集合中的元素，key和value的数据类型**可以相同，也可以不同**

3. Map集合中的元素，**key是不允许重复的，value是可以重复的**

4. Map集合中的元素，key和value是**一一对应的**

   

#### 数据结构

HashMap集合底层是哈希值：查询的速度特别的快。

1. JDK.8之前：数组+单向链表

2. JDK.8之后：数组+单向链表/红黑树**（链表的长度超过8）**：提高查询的速度

   

#### Entry键值对对象

Map.Entry<K,V>:在Map接口中有一个**内部接口Entry**
**作用：**当Map集合一创建，那么就会在Map集合中创建一个Entry对象，用来记录键与值（键值对对象,键与值的映射关系)

#### key判断

只有equals()和hashcode()两个相等的时候， 才表示key是一样的。 



#### 总结

1. HashMap采用数组方式存储key、value构成的Entry对象，无容量限制；
2. HashMap基于key hash寻找Entry对象存放到数组的位置，对于hash冲突采用链表的方式解决；
3. HashMap在插入元素时可能会扩大数组的容量，在扩大容量时须要重新计算hash，并复制对象到新的数组中；
4. **HashMap是非线程安全的。**
5. 不过HashMap有一个问题，就是**迭代HashMap的顺序并不是HashMap放置的顺序**，也就是无序。



#### 子类--LinkedHashMap 

转载地址：https://www.cnblogs.com/ericguoxiaofeng/p/10326014.html 

##### 概述

LinkedHashMap 在 HashMap 的基础上单独维护了一个具有所有数据的双向链表，该链表保证了元素迭代的顺序。

所以我们可以直接这样说：LinkedHashMap = HashMap + LinkedList。LinkedHashMap 就是在 HashMap 的基础上多维护了一个双向链表，用来保证元素迭代顺序。

***所以在概念上面说，比hashmap多了before，after，head，tail这几个东东***

##### 特点

| 关 注 点                        | 结 论                        |
| ------------------------------- | ---------------------------- |
| LinkedHashMap是否允许键值对为空 | Key和Value都允许空           |
| LinkedHashMap是否允许重复数据   | Key重复会覆盖、Value允许重复 |
| LinkedHashMap是否有序           | 有序                         |
| LinkedHashMap是否线程安全       | 非线程安全                   |



### TreeMap

转载地址： https://blog.csdn.net/aa1215018028/article/details/90477177

#### 概述

TreeMap基于**红黑树（Red-Black tree）实现**。该映射根据**其键的自然顺序进行排序**，或者根据**创建映射时提供的 Comparator 进行排序**，具体取决于使用的构造方法。

TreeMap的基本操作 containsKey、get、put 和 remove 的时间复杂度是 log(n) 。

另外，TreeMap是**非同步**的。 它的iterator 方法返回的**迭代器是fail-fastl**的。



#### 特点

1. TreeMap 是一个**有序的key-value集合**，它是通过[红黑树](http://www.cnblogs.com/skywang12345/p/3245399.html)实现的。

2. TreeMap **继承于AbstractMap**，所以它是一个Map，即一个key-value集合。

3. TreeMap 实现了NavigableMap接口，意味着它**支持一系列的导航方法。**比如返回有序的key集合。

4. TreeMap 实现了Cloneable接口，意味着**它能被克隆**。

5. TreeMap 实现了java.io.Serializable接口，意味着**它支持序列化**。

6. TreeMap只能根据key来排序，是不能根据value来排序的

   

#### 源码查看

```java
 public TreeMap() {
        comparator = null;
    }
    
 public TreeMap(Comparator<? super K> comparator) {
        this.comparator = comparator;
    }   
```

提供排序的接口：Comparator



### WeakHashmap

转载地址：https://blog.csdn.net/m0_37564531/article/details/88042987

#### 概述

WeakHashMap也是Map中的一种，它的特殊之处在于它其中的对象，可能会被GC自动回收，也就是数据会凭空消失，即使程序员没有调用remove()或者clean()方法，那么你就会有疑问了，这种数据会凭空消失，那为什么还要设计这种类？其实答案是WeakHashMap主要用于缓存的情况，缓存一般用于内存不够的情况下，设置缓存可以大大提高系统的效率，缓存的空间也是有限的所以说，WeakHashMap用来存储需要缓存的对象，如果命中了当然是很好的，如果没命中也无关紧要，而且正是这种自动回收不需要(GC认为不需要)的数据，反而能帮我们管理缓存的空间。

#### 特点 

1. WeakHashMap的初始容量为16，且必须为2的次幂
2. WeakHashMap的最大容量为2的30次方
3. 负载因子为0.75
4. WeakHashMap是线程不安全的



### EnumMap

转载地址：https://blog.csdn.net/chinakite/article/details/3244934

#### 概述

EnumMap是专门为枚举类型量身定做的Map实现。虽然使用其它的Map实现（如HashMap）也能完成枚举类型实例到值得映射，但是使用EnumMap会更加高效：它只能接收同一枚举类型的实例作为键值，并且由于枚举类型实例的数量相对固定并且有限，所以EnumMap使用数组来存放与枚举类型对应的值。这使得EnumMap的效率非常高。

#### 代码示例

```java
//现支持的数据库类型枚举类型定义

public enum DataBaseType{
     MYSQL,ORACLE,DB2,SQLSERVER
}

//某类中定义的获取数据库URL的方法以及EnumMap的声明。
private EnumMap<DataBaseType ,String> urls = new EnumMap<DataBaseType ,String>(DataBaseType.class);


public DataBaseInfo(){
         urls.put(DataBaseType.DB2,"jdbc:db2://localhost:5000/sample");
         urls.put(DataBaseType.MYSQL,"jdbc:mysql://localhost/mydb");
         urls.put(DataBaseType.ORACLE,"jdbc:oracle:thin:@localhost:1521:sample");
        urls.put(DataBaseType.SQLSERVER,"jdbc:microsoft:sqlserver://localhost:1433;DatabaseName=mydb");

}

/**

* 根据不同的数据库类型，返回对应的URL

* @param type     DataBaseType枚举类新实例

* @return

*/

public String getURL(DataBaseType type){
         return this.urls.get(type);
}
```



### IdentityHashMap

#### 概述

在[Java](http://lib.csdn.net/base/java)中，有一种key值可以重复的map，就是IdentityHashMap。在IdentityHashMap中，判断两个键值k1和 k2相等的条件是 k1 == k2 。在正常的Map 实现（如 HashMap）中， 要比较equals和hascode方法，都相等时候才相等。 

**而我们的IdentityHashMap，比较key值，直接使用的是==**

#### == 和 equals比较

![20180602183200272](/img//20180602183200272.png)

#### 代码示例

```java
public class CollectionTest {

    public static void main(String[] args) {

        // IdentityHashMap使用===================================
        Map<Person, String> hashmap = new HashMap<>();
        Person pp1 = new Person(1);
        Person pp2 = new Person(1);
        Person pp3 = new Person(1);
        hashmap.put(pp1, "1");
        hashmap.put(pp2, "2");
        hashmap.put(pp3, "3");
        System.out.println(hashmap.size()); // 3

        Map<Person, String> identityHashMap2 = new IdentityHashMap<>();
        Person p1 = new Person(1);
        Person p2 = new Person(1);
        Person p3 = new Person(1);
        identityHashMap2.put(p1, "1");
        identityHashMap2.put(p2, "2");
        identityHashMap2.put(p3, "3");
        System.out.println(identityHashMap2.size()); // 3

        System.out.println("--------------------------------" + identityHashMap2.get(p1));

    }
}

class Person {

    public String name;
    public int age;

    public Person(int age) {
        this.age = age;
    }

    @Override
    public int hashCode() {
        return 0;
    }

    @Override
    public boolean equals(Object obj) {
        return true;
    }

    @Override
    public String toString() {
        return this.name + ":" + this.age;
    }

}
运行结果
1
3
--------------------------------1
```

#### 特点

1. IdentityHashMap也是无序的
2. 该类不是线程安全的。



### HashTable

#### 概述

HashTable数据结构的原理大致一样，区别在于put、get时加了同步关键字，**而且HashTable不可存放null值。**

##### 子类--Dictionary

###### 概述

Dictionary 类是一个抽象类，用来存储键/值对，作用和Map类相似。**此类已过时。新的实现应该实现 Map 接口，而不是扩展此类。**



### ConcurrentHashMap

转载地址：https://baijiahao.baidu.com/s?id=1617089947709260129&wfr=spider&for=pc 

#### 概述

ConcurrentHashMap是Java中的一个**线程安全且高效的HashMap实现**。平时涉及高并发如果要用map结构，那第一时间想到的就是它。

#### 为什么用它

1. 我们知道HashMap是线程不安全的，在多线程环境下，使用Hashmap进行put操作会引起死循环，导致CPU利用率接近100%，所以在并发情况下不能使用HashMap。
2. HashTable线程安全的策略实现代价却太大了，简单粗暴，get/put所有相关操作都是synchronized的，这相当于给整个哈希表加了一把大锁。多线程访问时候，只要有一个线程访问或操作该对象，那其他线程只能阻塞，相当于将所有的操作串行化，在竞争激烈的并发场景中性能就会非常差。

主要就是为了应对hashmap在并发环境下不安全而诞生的，ConcurrentHashMap的设计与实现非常精巧，大量的利用了volatile，final，CAS等lock-free技术来减少锁竞争对于性能的影响。



### ConcurrentSkipListMap

转载地址：https://www.cnblogs.com/java-zzl/p/9767255.html 

#### 概述

ConcurrentSkipListMap是线程安全的有序的哈希表，适用于高并发的场景。ConcurrentSkipListMap和TreeMap，它们虽然都是有序的哈希表。但是，第一，它们的线程安全机制不同，TreeMap是非线程安全的，而ConcurrentSkipListMap是线程安全的。第二，ConcurrentSkipListMap是通过跳表实现的，而TreeMap是通过红黑树实现的。

#### ConcurrentSkipListMap VS ConcurrentHashMap 不能比拟的优点

1. ConcurrentSkipListMap 的key是有序的 
2. ConcurrentSkipListMap 支持更高的并发。ConcurrentSkipListMap 的存取时间是log（N），和线程数几乎无关。也就是说在数据量一定的情况下，并发的线程越多，ConcurrentSkipListMap越能体现出他的优势。

#### 总结

在非多线程的情况下，应当尽量使用TreeMap。此外对于并发性相对较低的并行程序可以使用Collections.synchronizedSortedMap将TreeMap进行包装，也可以提供较好的效率。对于高并发程序，应当使用ConcurrentSkipListMap，能够提供更高的并发度。

# Comparable接口

原文链接：https://blog.csdn.net/nvd11/article/details/27393445

## 概述

此接口强行对实现它的**每个类的对象进行整体排序**。这种排序被称为类的*自然排序*，类的 compareTo 方法被称为它的*自然比较方法*。

总而言之,  如果你想1个类的对象支持比较(排序), 就必须实现Comparable接口.

## Comparable接口简介

Comparable 接口内部只有1个要重写的关键的方法.

就是

```java
int compareTo(T o)
```

这个方法返回1个Int数值,  例如 i = x.compareTo(y)

如果i=0, 也表明对象x与y排位上是相等的(并非意味x.equals(y) = true, 但是jdk api上强烈建议这样处理)

如果返回数值i>0 则意味者, x > y啦，　

反之若i<0则　意味x < y

## 实例显示

```java
public class CollectionTest {

    public static void main(String[] args) {
        List<Person> persons = new ArrayList<Person>();
        Person person = new Person();
        person.name = "小米-麦子";
        person.age = 30;
        persons.add(person);

        for (int i = 0; i < 10; i++) {
            person = new Person();
            person.name = "小米-" + i;
            person.age = i + 1;
            persons.add(person);
        }
        Collections.sort(persons);
        persons.forEach(System.out::println);
    }
}

class Person implements Comparable<Person> {

    public String name;
    public int age;

    @Override
    public int compareTo(Person o) {

        return this.age - o.age;
    }

    @Override
    public String toString() {
        return this.name + ":" + this.age;
    }

}
```

运行结果

```java
小米-0:1
小米-1:2
小米-2:3
小米-3:4
小米-4:5
小米-5:6
小米-6:7
小米-7:8
小米-8:9
小米-9:10
```

 

## 注意

我们常用的String， Integer等类，都是实现了这个接口， 所以在Arrays.sort()排序的时候可以直接使用。 



# Iterator接口

转载地址：https://www.cnblogs.com/Exton-Learning/articles/10351260.html 

## 概述

Iterator接口也是Java集合框架的成员，但它与Collection系列、Map系列的集合不一样：Collection系列集合、Map系列集合主要用于盛装其他对象，而Iterator则主要用于遍历（即迭代访问）Collection集合中的元素，Iterator对象也被称为迭代器。**迭代器是一种设计模式,具体查看博客《迭代器设计模式》**

**迭代器是一种设计模式，它是一个对象，可以遍历并选择序列中的对象，而开发人员不需要了解该序列的底层结构。迭代器通常器被称为“轻量级”对象，因为创建它的代价小。**

**Iterator的一个很强的用处：能够将遍历序列的操作与序列底层的结构分离。**

## 常用方法

Iterator接口里定义了如下4个方法：

1. boolean hasNext()：如果被迭代的集合还元素没有被遍历，则返回true。
2. Object next()：返回集合里下一个元素。
3. void remove() ：删除集合里上一次next方法返回的元素
4. void forEachRemaining(Consumer action)，这是Java 8为Iterator新增的默认方法，该方法可使用Lambda表达式来遍历集合元素

## 显示结构

![2018062816262240](/img//2018062816262240.png)

### remove方法

一开始迭代器**在所有元素的左边**，调用next()之后，**迭代器移到第一个和第二个元素之间**，next()方法**返回迭代器刚刚经过的元素**。
hasNext()若**返回True，则表明接下来还有元素**，迭代器不在尾部。
remove()方法必须和next方法一起使用，功能是**去除刚刚next方法返回的元素**。

### 注意

iterator()方法是java.lang.Iterable接口,被Collection继承.

```java
public interface Collection<E> extends Iterable<E> {}

public interface Iterable<T> {
    /**
     * Returns an iterator over elements of type {@code T}.
     *
     * @return an Iterator.
     */
    Iterator<T> iterator();
 }   
```



### 相关子类

#### ListIterator

##### 局限

只能应用于各种List类的访问。

##### 优势

1. Iterator只能向前移动，而ListIterator可以双向移动。
2. 还可以产生相对于迭代器在列表中指向的当前位置的前一个和后一个元素的索引nextIndex()、previousIndex()方法。
3. 还可以通过set（）方法替换它访问过的最后一个元素。
4. 还可以通过调用listIterator（）方法产生一个指向List开始处的ListIterator，当然也可以有参数，即指向索引为参数处的ListIterator。
5. ListIterator 有 add() 方法，可以向 List 中添加对象，而 Iterator 不能。

# 工具类

## Collections  

转载地址：https://www.cnblogs.com/ZeroMZ/p/11402577.html

### 概述

   工具类中提供的方法主要针对Set、List、Map的排序、查询、修改等操作，以及将集合对象设置为不可变、对集合对象实现同步控制（线程安全）等方法。

### 主要方法

#### 排序

![1242064-20190823203501488-926112856](/img//1242064-20190823203501488-926112856.png)

#### 查找，替换

![1242064-20190823203638404-641551560](/img//1242064-20190823203638404-641551560.png)

#### 同步控制

Collections类中提供了多个synchronizedXxx()方法，这些方法可以将指定集合包装成线程同步的集合，从而可以解决多线程并发访问集合时的线程安全问题。

Java中常用的实现类HashSet、TreeSet、ArrayList、ArrayDeque、LinkedList、HashMap、TreeMap共**7**种集合都是线程不安全的。

```java
//通过以下的包装，就可以实现线程安全的集合对象
Collection c= Collections.synchronizedCollection(new ArrayList());
List list=Collections.synchronizedList(new ArrayList());
Set s=Collections.synchronizedSet(new HashSet());
Map m=Collections.synchronizedMap(new HashMap());
```

#### 设置不可变集合

![1242064-20190823204851112-2089645757](/img//1242064-20190823204851112-2089645757.png)

上面三类方法的参数是原有的集合对象，返回值是该集合的“只读”版本。即无论集合中的元素个数、还是集合中的元素的修改，都会引发错误。



## Arrays

转载地址：https://blog.csdn.net/Goodbye_Youth/article/details/81003817 

### 概述

Arrays类位于 **java.util** 包中，主要包含了操纵数组的各种方法。 

### 常用方法

**1.  Arrays.asList(T… data)**

```java
注意：该方法返回的是 Arrays 内部静态类 ArrayList，而不是我们平常使用的 ArrayList,，该静态类 ArrayList 没有覆盖父类的 add(), remove() 等方法，所以如果直接调用，会报 UnsupportedOperationException 异常
 
List<Integer> list = Arrays.asList(1, 2, 3);
list.forEach(System.out::println); // 1 2 3
```
 **2.  Arrays.fill(Object[] array, Object obj)**

```java
用指定元素填充整个数组 (会替换掉数组中原来的元素)

Integer[] data = {1, 2, 3, 4};
Arrays.fill(data, 9);
System.out.println(Arrays.toString(data)); // [9, 9, 9, 9]
```

**3.  Arrays.fill(Object[] array, int fromIndex, int toIndex, Object obj)**

```java
用指定元素填充数组，从起始位置到结束位置，取头不取尾 (会替换掉数组中原来的元素)

Integer[] data = {1, 2, 3, 4};
Arrays.fill(data, 0, 2, 9);
System.out.println(Arrays.toString(data)); // [9, 9, 3, 4]
```

**4.  Arrays.sort(Object[] array)**

```java
对数组元素进行排序 (串行排序)

String[] data = {"1", "4", "3", "2"};
System.out.println(Arrays.toString(data)); // [1, 4, 3, 2]
Arrays.sort(data);
System.out.println(Arrays.toString(data)); // [1, 2, 3, 4]

// 使用自定义比较器，对数组元素进行排序 (串行排序)
Arrays.sort(T[] array, Comparator<? super T> comparator)
  
// 对指定范围内的数组元素进行排序 (串行排序)
Arrays.sort(Object[] array, int fromIndex, int toIndex)
  
Arrays.sort(T[] array, int fromIndex, int toIndex, Comparator<? super T> c)
```

**5.  Arrays.parallelSort(T[] array)**

```java
注意：其余重载方法与 Arrays.sort() 相同
对数组元素进行排序 (并行排序)，当数据规模较大时，会有更好的性能

String[] data = {"1", "4", "3", "2"};
Arrays.parallelSort(data);
System.out.println(Arrays.toString(data)); // [1, 2, 3, 4]
```

**6. Arrays.binarySearch(Object[] array, Object key)**

```java
注意：在调用该方法之前，必须先调用 Arrays.sort() 方法进行排序，如果数组没有排序，那么结果是不确定的，此外如果数组中包含多个指定元素，则无法保证将找到哪个元素

使用二分法查找数组内指定元素的索引值

// 使用二分法查找数组内指定范围内的指定元素的索引值 
Arrays.binarySearch(Object[] array, int fromIndex, int toIndex, Object obj)

Integer[] data = {1, 3, 5, 7};
Arrays.sort(data);
// {1, 3}，3的索引值为1
System.out.println(Arrays.binarySearch(data, 0, 2, 3)); // 1

```

**7.  Arrays.copyOf(T[] original, int newLength)**

```java
拷贝数组，其内部调用了 System.arraycopy() 方法，从下标 0 开始，如果超过原数组长度，则会用 null 进行填充

Integer[] data1 = {1, 2, 3, 4};
Integer[] data2 = Arrays.copyOf(data1, 2);
System.out.println(Arrays.toString(data2)); // [1, 2]
Integer[] data3 = Arrays.copyOf(data1, 5);
System.out.println(Arrays.toString(data3)); // [1, 2, 3, 4, null]
```

**8.  Arrays.toString(Object[] array)**

```java
返回数组元素的字符串形式

Integer[] data = {1, 2, 3};
System.out.println(Arrays.toString(data)); // [1, 2, 3]
```

**9.  Arrays.setAll(T[] array, IntFunction<? extends T> generator)**

```java
让数组中的所有元素，串行地使用方法提供的生成器函数来计算每个元素 (一元操作)

Integer[] data = {1, 2, 3, 4};
// i为索引值
Arrays.setAll(data, i -> data[i] * 2);
System.out.println(Arrays.toString(data)); // [2, 4, 6, 8]
```

**10. Arrays.parallelSetAll(T[] array, IntFunction<? extends T> generator)**

```java
让数组中的所有元素，并行地使用方法提供的生成器函数来计算每个元素 (一元操作)，当数据规模较大时，会有更好的性能

Integer[] data = {1, 2, 3, 4};
// i为索引值
Arrays.parallelSetAll(data, i -> data[i] * 2);
System.out.println(Arrays.toString(data)); // [2, 4, 6, 8]
```

**11. Arrays.spliterator(T[] array)**

```java
返回数组的分片迭代器，用于并行地遍历数组
```

**12. Arrays.stream(T[] array)**

```java
返回数组的流 (Stream)，然后我们就可以使用 Stream 相关的许多方法了

Integer[] data = {1, 2, 3, 4};
List<Integer> list = Arrays.stream(data).collect(toList());
System.out.println(list); // [1, 2, 3, 4]
```



## 集合类互相转换

转载地址：https://blog.csdn.net/moye666/article/details/93601135 

#### List与Set互相转换

##### list转set

```java
// 这个转换通常用来做去重 

List<String> arrayList = new ArrayList<>();
arrayList.add("a");
arrayList.add("b");
arrayList.add("c");
arrayList.add("a");
Set<String> newHashSet = new HashSet<>(arrayList);

System.out.println("arrayList:" + arrayList.toString());
System.out.println("newHashSet:" + newHashSet.toString());
```

##### set转list

```java
Set<String> hashSet = new HashSet<>();
hashSet.add("a");
hashSet.add("c");
hashSet.add("b");
hashSet.add("c");
hashSet.add("d");
List<String> newArrayList = new ArrayList<>(hashSet);
List<String> newLinkedList = new LinkedList<>(hashSet);
```



#### Map转List，Set

```java
Map<Integer, Person> map = new HashMap<Integer, Person>();
Set<Entry<Integer, Person>> data = map.entrySet();
List<Entry<Integer, Person>> listdata = new ArrayList<Entry<Integer, Person>>(data);
```



#### 注意

**需要更多的去选择用流来进行集合类的转换。** 



# 常见问题

## 1. LinkedList和ArrayList的对比

转载地址： https://www.cnblogs.com/xiaoxi/p/6102220.html

有些说法认为LinkedList做插入和删除更快，这种说法其实是不准确的：

1. LinkedList做插入、删除的时候，慢在寻址，快在只需要改变前后Entry的引用地址

2. ArrayList做插入、删除的时候，慢在数组元素的批量copy，快在寻址。同时删除时候要把数据往前移动数据

所以，如果待插入、删除的元素是在数据结构的前半段尤其是非常靠前的位置的时候，LinkedList的效率将大大快过ArrayList，因为ArrayList将批量copy大量的元素；越往后，对于LinkedList来说，因为它是双向链表，所以在第2个元素后面插入一个数据和在倒数第2个元素后面插入一个元素在效率上基本没有差别，但是ArrayList由于要批量copy的元素越来越少，操作速度必然追上乃至超过LinkedList。

从这个分析看出，如果你十分确定你插入、删除的元素是在前半段，那么就使用LinkedList；如果你十分确定你删除、删除的元素在比较靠后的位置，那么可以考虑使用ArrayList。如果你不能确定你要做的插入、删除是在哪儿呢？那还是建议你使用LinkedList吧，因为一来LinkedList整体插入、删除的执行效率比较稳定，没有ArrayList这种越往后越快的情况；二来插入元素的时候，弄得不好ArrayList就要进行一次扩容，记住，**ArrayList底层数组扩容是一个既消耗时间又消耗空间的操作。**



## 2. CopyOnWriteArrayList为什么并发安全且性能比Vector好

我知道Vector是增删改查方法都加了synchronized，保证同步，但是每个方法执行的时候都要去获得锁，性能就会大大下降，而CopyOnWriteArrayList 只是在增删改上加锁，但是读不加锁，在读方面的性能就好于VectorCopyOnWriteArrayList支持读多写少的并发情况。



## 3. HashSet，TreeSet和LinkedHashSet的区别

总体而言，如果你需要一个访问快速的Set，你应该使用HashSet；当你需要一个排序的Set，你应该使用TreeSet；当你需要记录下插入时的顺序时，你应该使用LinedHashSet。

### HashSet

1. 不能保证元素的排列顺序，顺序有可能发生变化
2. 不是同步的
3. 集合元素可以是null,但只能放入一个null(注：List可以添加多个null对象。)

### LinkedHashSet

LinkedHashSet在迭代访问Set中的全部元素时，性能比HashSet好，但是插入时性能稍微逊色于HashSet。



## 4.  LinkedHashMap VS TreeMap

LinkedHashMap将按照将条目放入地图的顺序进行迭代。

TreeMap将根据密钥的“自然顺序”按其

```
compareTo()
```

方法(或外部提供的

```
Comparator
```

)。此外，它还实现了SortedMap 接口，它包含依赖于此排序顺序的方法。



## 5. set排序 ， List排序， map排序

#### 示例公共代码

```java
class Person {

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

}
```



#### List排序

```java
    public static void main(String[] args) {
        List<Person> persionList = new ArrayList<Person>();
        Person pp1 = new Person("小米",10);
        Person pp2 = new Person("小花",2);
        Person pp3 = new Person("小猫",5);
        persionList.add(pp1);
        persionList.add(pp2);
        persionList.add(pp3);

        Collections.sort(persionList, new Comparator<Person>(){

                 @Override
                 public int compare(Person o1, Person o2) {
                     
                     return o1.age - o2.age;
                 }
        });

        persionList.forEach(System.out::println);
    }
}
```



#### Set排序

```java
 public static void main(String[] args) {
 
        Set<Person> persionList = new HashSet<Person>();
        Person pp1 = new Person("小米",10);
        Person pp2 = new Person("小花",2);
        Person pp3 = new Person("小猫",5);
        persionList.add(pp1);
        persionList.add(pp2);
        persionList.add(pp3);


        TreeSet<Person> set = new TreeSet<Person>(new Comparator<Person>(){
              @Override
              public int compare(Person o1, Person o2) {
                  
                  return o1.age - o2.age;
              }
        });
        set.addAll(persionList);
        set.forEach(System.out::println);
    }
```



#### Map排序

##### key排序

```java
 public static void main(String[] args) {

        Map<Integer, Person> map = new HashMap<Integer, Person>();
        Person pp1 = new Person("小米", 10);
        Person pp2 = new Person("小花", 2);
        Person pp3 = new Person("小猫", 5);
        map.put(10, pp1);
        map.put(1, pp2);
        map.put(2, pp3);

        TreeMap<Integer, Person> treeMap = new TreeMap<Integer, Person>(new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {

                return o1 - o2;
            }
        });
        treeMap.putAll(map);

        Iterator iter = map.entrySet().iterator();
        while (iter.hasNext()) {
            Map.Entry<Integer, Person> entry = (Map.Entry<Integer, Person>) iter.next();
            int key = entry.getKey();
            Person person = entry.getValue();
            System.out.println(key + "--->" + person);
        }

    }
```

##### value排序

大致的思路是把TreeMap的EntrySet转换成list，然后使用Collections.sor排序。

```java
   public static void main(String[] args) {

        Map<Integer, Person> map = new HashMap<Integer, Person>();
        Person pp1 = new Person("小米", 10);
        Person pp2 = new Person("小花", 20);
        Person pp3 = new Person("小猫", 5);
        map.put(10, pp1);
        map.put(1, pp2);
        map.put(2, pp3);

        List<Entry<Integer, Person>> list = new ArrayList<Entry<Integer, Person>>(map.entrySet());
        Collections.sort(list, new Comparator<Entry<Integer, Person>>() {

            @Override
            public int compare(Entry<Integer, Person> o1, Entry<Integer, Person> o2) {
                Person p1 = o1.getValue();
                Person p2 = o2.getValue();
                return p1.age - p2.age;
            }

        });

        list.forEach(System.out::println);

    }
```



### 6. HashMap怎么存放大量数据。 
---
title:       "equals,hasCode,toString"
subtitle:    ""
description: ""
date:        2019-06-24
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "equals,hasCode,toString"]
categories:  ["Tech" ]
---

[TOC]

# HashCode有什么用

**转载地址：https://www.cnblogs.com/xrq730/p/4842028.html**

回到最关键的问题，HashCode有什么用？不妨举个例子：

1、假设内存中有0 1 2 3 4 5 6 7 8这9个位置，如果我有个字段叫做ID，那么我要把这个字段存放在以上9个位置之一，如果不用HashCode而任意存放，那么当查找时就需要到8个位置中去挨个查找

2、使用HashCode则效率会快很多，把ID的HashCode % 9，然后把ID存放在取得余数的那个位置，然后每次查找该类的时候都可以通过ID的HashCode % 9求余数直接找到存放的位置了

3、如果ID的HashCode % 9算出来的位置上本身已经有数据了怎么办？这就取决于算法的实现了，比如ThreadLocal中的做法就是从算出来的位置向后查找第一个为空的位置，放置数据；HashMap的做法就是通过链式结构连起来。反正，只要保证放的时候和取的时候的算法一致就行了。

4、如果ID的HashCode % 9相等怎么办（这种对应的是第三点说的链式结构的场景）？这时候就需要定义equals了。先通过HashCode%8来判断类在哪一个位置，再通过equals来在这个位置上寻找需要的类。对比两个类的时候也差不多，先通过HashCode比较，假如HashCode相等再判断equals。**如果两个类的HashCode都不相同，那么这两个类必定是不同的**。

举个实际的例子Set：

我们知道Set里面的元素是不可以重复的，那么如何做到？Set是根据equals()方法来判断两个元素是否相等的。比方说Set里面已经有1000个元素了，那么第1001个元素进来的时候，最多可能调用1000次equals方法，如果equals方法写得复杂，对比的东西特别多，那么效率会大大降低。使用HashCode就不一样了，比方说HashSet，底层是基于HashMap实现的，先通过HashCode取一个模，这样一下子就固定到某个位置了，如果这个位置上没有元素，那么就可以肯定HashSet中必定没有和新添加的元素equals的元素，就可以直接存放了，都不需要比较；如果这个位置上有元素了，逐一比较，比较的时候先比较HashCode，HashCode都不同接下去都不用比了，肯定不一样，HashCode相等，再equals比较，没有相同的元素就存，有相同的元素就不存。如果原来的Set里面有相同的元素，只要HashCode的生成方式定义得好（不重复），不管Set里面原来有多少元素，只需要执行一次的equals就可以了。这样一来，实际调用equals方法的次数大大降低，提高了效率。

**之所以有hashCode方法，是因为在批量的对象比较中，hashCode要比equals来得快，很多集合都用到了hashCode，比如HashTable。**

**java的HashCode方法：**

set中，要想保证元素不重复，需要依据Object.equals方法，但是，如果每增加一个元素就检查一次，那么当元素很多时，后添加到集合中的元素比较的次数就非常多了。

**hashCode方法实际上返回的就是对象存储的物理地址（实际可能并不是）**。   这样一来，当集合要添加新的元素时，先调用这个元素的hashCode方法，就一下子能定位到它应该放置的物理位置上。 如果这个位置上没有元素，它就可以直接存储在这个位置上，不用再进行任何比较了；如果这个位置上已经有元素了， 就调用它的equals方法与新元素进行比较，相同的话就不存了，不相同就散列其它的地址。 所以这里存在一个冲突解决的问题。这样一来实际调用equals方法的次数就大大降低了，几乎只需要一两次。   所以，Java对于eqauls方法和hashCode方法是这样规定的：

1. 如果两个对象相同，那么它们的hashCode值一定要相同；
2. 如果两个对象的hashCode相同，它们并不一定相同;

# 为什么要重写equals方法

**转载地址：https://www.cnblogs.com/jesonjason/p/5492208.html**

因为Object的equal方法默认是两个对象的引用的比较，意思就是指向同一内存,地址则相等，否则不相等；如果你现在需要利用对象里面的值来判断是否相等，则重载equal方法。

说道这个地方我相信很多人会有疑问，相信大家都被String对象的equals()方法和"=="纠结过一段时间，当时我们知道String对象中equals方法是判断值的，而==是地址判断。

那照这么说equals怎么会是地址的比较呢？

那是因为实际上JDK中，String、Math等封装类都对Object中的equals()方法进行了重写。

我们先看看Object中equals方法的源码：

```java
public boolean equals(Object obj) { 
         return (this == obj); 
}
```

我们都知道所有的对象都拥有标识(内存地址)和状态(数据)，同时“==”比较两个对象的的内存地址，所以说使用Object的equals()方法是比较两个对象的内存地址是否相等，即若object1.equals(object2)为true，则表示equals1和equals2实际上是引用同一个对象。虽然有时候Object的equals()方法可以满足我们一些基本的要求，但是我们必须要清楚我们很大部分时间都是进行两个对象的比较，这个时候Object的equals()方法就不可以了，**所以才会有String这些类对equals方法的改写，依次类推Double、Integer、Math。。。。等等这些类都是重写了equals()方法的**，从而进行的是内容的比较。希望大家不要搞混了。

# 改写equals时总是要改写hashcode

又如new一个对象，再new一个内容相等的对象，调用equals方法返回的true，但他们的hashcode值不同，将两个对象存入HashSet中，会使得其中包含两个相等的对象，因为是先检索hashcode值，不等的情况下才会去比较equals方法的。

```java
@Setter
@Getter
@AllArgsConstructor
public class Student {

    private  int id;
    private  String name;

    // @Override
    // public int hashCode() {
    //     final int prime = 31;
    //     int result = 1;
    //     result = prime * result + id;
    //     result = prime * result + ((name == null) ? 0 : name.hashCode());
    //     return result;
    // }

    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;
        Student other = (Student) obj;
        if (id != other.id)
            return false;
        if (name == null) {
            if (other.name != null)
                return false;
        } else if (!name.equals(other.name))
            return false;
        return true;
    }


    public static void main(String[] args) {
        Student student = new Student(1,"小强");
        Student student2 = new Student(1,"小强");
        System.out.println(student.equals(student2));
        
        System.out.println(student.hashCode());
        System.out.println(student2.hashCode());

        System.out.println(student == student2);


        HashSet<Student> set = new HashSet<Student>();
        set.add(student);
        set.add(student2);
        System.out.println(set.size()); // 这样的话就存入了2个相同的对象了
    }
    
}
```

运行结果如下， HashSet中存入了两个相同的值了，** size为2了。**

```java
true
2018699554
1311053135
false
2
```

**在改写equals的时候总是要改写hashCode，如果不这样的话，就会违反Object.hashCode的通用约定，导致这个类无法与所有基于散列值的集合类结合在一起正常工作，包括HashMap,HashSet和Hashtable。**

# toString()方法

因为toString方法是Object里面已经有了的方法，而所有类都是继承Object，所以“所有对象都有这个方法”。

它通常只是为了方便输出，比如System.out.println(xx)，括号里面的“xx”如果不是String类型的话，就自动调用xx的toString()方法

总而言之，它只是sun公司开发java的时候为了方便所有类的字符串操作而特意加入的一个方法
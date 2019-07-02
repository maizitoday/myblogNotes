---
title:       "Jdk1.8新特性-方法引用"
subtitle:    ""
description: ""
date:        2019-07-02
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "方法引用"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://blog.csdn.net/wsgytwsgyt/article/details/80273678**

# 方法引用定义

**方法引用是lambda表达式的一种特殊形式**，如果正好有某个方法满足一个lambda表达式的形式，那就可以将这个lambda表达式用方法引用的方式表示，但是如果这个lambda表达式的比较复杂就不能用方法引用进行替换。

若Lambda体重的内容有方法以及实现了，我们可以使用"方法引用"。可以理解方法引用是lambda表达式的另外一种表现形式。

# 方法引用共分为四类



## 类名::静态方法名

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Student {
    private String name;
    private int score;


    public static int compareStudentByScore(Student student1,Student student2){
        return student1.getScore() - student2.getScore();  
    }

    public static void main(String[] args) {
        Student student1 = new Student("zhangsan",100);
        Student student2 = new Student("lisi",70);
        Student student3 = new Student("wangwu",80);
        Student student4 = new Student("zhaoliu",90);

        List<Student> dataList = Arrays.asList(student1,student2,student3,student4);
        // dataList.sort((o1,o2) -> o1.getScore() - o2.getScore());
        // dataList.forEach(student -> System.out.println(student.getScore()));

        // 同样是接收两个参数返回一个int类型值，而且是对Student对象的分数进行比较，
        // 所以我们这里就可以 使用类名::静态方法名 方法引用替换lambda表达式
        dataList.sort(Student::compareStudentByScore);
        dataList.forEach(student -> System.out.println(student.getScore()));
    }
}
```



## 对象::实例方法名

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Student {
    private String name;
    private int score;

    // 同样该方法的定义满足Comparator接口的compare方法定义，
    // 所以这里可以直接使用 对象::实例方法名 的方式使用方法引用来替换lambda表达式
    public int compare(Student student1,Student student2){
        return student1.getScore() - student2.getScore();  
    }


    public static void main(String[] args) {
        Student student1 = new Student("zhangsan", 100);
        Student student2 = new Student("lisi", 70);
        Student student3 = new Student("wangwu", 80);
        Student student4 = new Student("zhaoliu", 90);

        List<Student> dataList = Arrays.asList(student1, student2, student3, student4);

        Student studentSort = new Student();
        dataList.sort(studentSort::compare);
        dataList.forEach(x ->System.out.println(x));
        
    }
```



## 类名::实例方法名 

```java
public static int compareStudentByScore(Student student1,Student student2){
    return student1.getScore() - student2.getScore();
}
```

虽然这个方法在语法上没有任何问题，可以作为一个工具正常使用，**但是有没有觉得其在设计上是不合适的或者是错误的。这样的方法定义放在任何一个类中都可以正常使用，而不只是从属于Student这个类**，那如果要定义一个只能从属于Student类的比较方法下面这个实例方法更合适一些

```java
public int compareByScore(Student student){
    return this.getScore() - student.getScore();
}
```

**接收一个Student对象和当前调用该方法的Student对象的分数进行比较即可**。现在我们就可以使用 类名::实例方法名 这种方式的方法引用替换lambda表达式了

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Student {
    private String name;
    private int score;
 
    public int compare(Student student2){
        return this.getScore() - student2.getScore();  
    }

    public static void main(String[] args) {
        Student student1 = new Student("zhangsan", 100);
        Student student2 = new Student("lisi", 70);
        Student student3 = new Student("wangwu", 80);
        Student student4 = new Student("zhaoliu", 90);

        List<Student> dataList = Arrays.asList(student1, student2, student3, student4);

        dataList.sort(Student::compare);
        dataList.forEach(x ->System.out.println(x));
        
    }

```

这里非常奇怪，sort方法接收的lambda表达式不应该是两个参数么，为什么这个实例方法只有一个参数也满足了lambda表达式的定义（想想这个方法是谁来调用的）。**这就是 类名::实例方法名 这种方法引用的特殊之处：当使用 类名::实例方法名 方法引用时，一定是lambda表达式所接收的第一个参数来调用实例方法，如果lambda表达式接收多个参数，其余的参数作为方法的参数传递进去。**

## 类名::new

也称构造方法引用，和前两种类似只要符合lambda表达式的定义即可，回想下Supplier函数式接口的get方法，不接收参数有返回值，正好符合无参构造方法的定义

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
// 当然你也可以定义多个参数的，函数式接口来满足多个参数创建对象
Supplier<Student> supplier = Student::new; 
System.out.println(supplier.get());
```


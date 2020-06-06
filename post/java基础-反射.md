---
title:       "反射"
subtitle:    ""
description: ""
date:        2019-07-10
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["java基础", "反射"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://www.cnblogs.com/yrstudy/p/6500982.html**

**转载地址：https://www.cnblogs.com/songanwei/p/9386749.html**

# 反射机制概述

Java 反射机制是在运行状态中，对于任意一个类，都能够获得这个类的所有属性和方法，对于任意一个对象都能够调用它的任意一个属性和方法。这种在运行时动态的获取信息以及动态调用对象的方法的功能称为Java 的反射机制。

Class 类与java.lang.reflect 类库一起对反射的概念进行了支持，该类库包含了Field,Method,Constructor类(每个类都实现了Member 接口)。这些类型的对象时由JVM 在运行时创建的，用以表示未知类里对应的成员。

这样你就可以使用Constructor 创建新的对象，用get() 和set() 方法读取和修改与Field 对象关联的字段，用invoke() 方法调用与Method 对象关联的方法。另外，还可以调用getFields() getMethods() 和 getConstructors() 等很便利的方法，以返回表示字段，方法，以及构造器的对象的数组。这样匿名对象的信息就能在运行时被完全确定下来，而在编译时不需要知道任何事情。

# Java反射机制

Java 反射机制是在运行状态中，对于任意一个类，都能够获得这个类的所有属性和方法，对于任意一个对象都能够 调用它的任意一个属性和方法。这种在运行时动态的获取信息以及动态调用对象的方法的功能称为 Java 的反射机 制。

Class 类与 java.lang.reﬂect 类库一起对反射的概念进行了支持，该类库包含了 Field,Method,Constructor 类 (每 个类都实现了 Member 接口)。这些类型的对象时由 JVM 在运行时创建的，用以表示未知类里对应的成员。

这样你就可以使用 Constructor 创建新的对象，用 get() 和 set() 方法读取和修改与 Field 对象关联的字段，用 invoke() 方法调用与 Method 对象关联的方法。另外，还可以调用 getFields() getMethods() 和 getConstructors() 等很便利的方法，以返回表示字段，方法，以及构造器的对象的数组。这样匿名对象的信息 就能在运行时被完全确定下来，而在编译时不需要知道任何事情。



# 为什么使用

我们为什么要使用反射，它的作用是什么，它在实际的编程中有什么应用。首先我们先明确两个概念，静态编译和动态编译。

**静态编译**：在编译时确定类型，绑定对象,即通过。 

**动态编译**：运行时确定类型，绑定对象。动态编译最大限度发挥了java的灵活性，体现了多    
　　态的应用，有以降低类之间的藕合性。

我们可以明确的看出动态编译的好处，而反射就是运用了动态编译创建对象。

# 反射好处

```java
interface fruit{  
    public abstract void eat();  
}  
    
class Apple implements fruit{  
    public void eat(){  
        System.out.println("Apple");  
    }  
}  
    
class Orange implements fruit{  
    public void eat(){  
        System.out.println("Orange");  
    }  
} 

// 构造工厂类  
// 也就是说以后如果我们在添加其他的实例的时候只需要修改工厂类就行了  
class Factory{  
    public static fruit getInstance(String fruitName){  
        fruit f=null;  
        if("Apple".equals(fruitName)){  
            f=new Apple();  
        }  
        if("Orange".equals(fruitName)){  
            f=new Orange();  
        }  
        return f;  
    }  
}  

class hello{  
    public static void main(String[] a){  
        fruit f=Factory.getInstance("Orange");  
        f.eat();  
    }  
    
}  
```

可以发现，每当我们要添加一种新的水果的时候，我们将不得不改变Factory中的源码，而往往改变原有正确代码是一种十分危险的行为。而且随着水果种类的增加，你会发现你的factory类会越来越臃肿，如果使用反射来处理如下：

```java
interface fruit{  
    public abstract void eat();  
}  
   
class Apple implements fruit{  
    public void eat(){  
        System.out.println("Apple");  
    }  
}  
   
class Orange implements fruit{  
    public void eat(){  
        System.out.println("Orange");  
    }  
}  
   
class Factory{  
    public static fruit getInstance(String ClassName){  
        fruit f=null;  
        try{  
            f=(fruit)Class.forName(ClassName).newInstance();  
        }catch (Exception e) {  
            e.printStackTrace();  
        }  
        return f;  
    }  
}  
class hello{  
    public static void main(String[] a){  
        fruit f=Factory.getInstance("Reflect.Apple");  
        if(f!=null){  
            f.eat();  
        }  
    }  
}
```

在出现新品种水果的时候，你完全不用去修改原有代码。

## 框架里面的用法

我们可以获取你在配置文件里面的写好的包的路径，然后我们自动在包下面找寻里面的所有的类，进行创建，这也是进行IOC化， 同时找里面的所有的注解等进行处理， 比如我们在spring框架中，通过找到@requestMapping里面的URL映射，通过URL和Controller的映射对应，如果一个请求过来了，我们就可以去找到是哪一个Controller，进一步定位调用哪一个方法。 

## 举一个看到过的例子

在实际开发中，我们需要把一个包中的class new出来，但是这个包中的类总是需要变动，那么怎么办，难道总是修改main方法中xxx=new xxx()吗。这样无疑是麻烦的。而运用反射。我们可以相应的增加一个配置文件，在里面记录包中所有的类名，包中类增加时就加一个类名，删除时就删除一个类名。让main方法去读取这个配置文件中的类名，通过反射获得实例，完全不用我们去修改main方法中的代码。

## 反射还有什么用那？

他甚至可以修改其他类中的私有属性。android开发中，我们需要改变一个私有标志位的时候，android源码并没有提供set方法，我们又不能改变源码，怎么办，反射可以完美解决这个问题。**可以修改私有属性。**

# 反射常用API



## 获取字节码的方式

在Java 中可以通过三种方法获取类的字节码(Class)对象

- 通过Object 类中的getClass() 方法，想要用这种方法必须要明确具体的类并且创建该类的对象。
- 所有数据类型都具备一个静态的属性.class 来获取对应的Class 对象。但是还是要明确到类，然后才能调用类中的静态成员。
- 只要通过给定类的字符串名称就可以获取该类的字节码对象，这样做扩展性更强。通过Class.forName() 方法完成，必须要指定类的全限定名，由于前两种方法都是在知道该类的情况下获取该类的字节码对象，因此不会有异常，但是Class.forName() 方法如果写错类的路径会报 ClassNotFoundException 的异常。

```java
public class ReflectTest {
    public static void main(String[] args) {
        Fruit fruit = new Fruit();

        // step 1
        System.out.println(fruit.getClass());

        // step 2
        System.out.println(Fruit.class);

        // step 3   
        Class fruitClass = null;
        try {
            // 加载进入了JVM中，具体看 java基础-Class.forName文章
            fruitClass = Class.forName("com.example.basedemo.reflectdemo.Fruit");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        System.out.println(fruitClass);
     }
}

运行结果：
class com.example.basedemo.reflectdemo.Fruit
class com.example.basedemo.reflectdemo.Fruit
class com.example.basedemo.reflectdemo.Fruit
```



## 反射机制获取类信息

### 不同构造器创建对象

通过反射机制创建对象，在创建对象之前要获得对象的构造函数对象，通过构造函数对象创建对应类的实例。

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Fruit {
     private int price;
     private String name;
     private String address;
    
}

 public static void main(String[] args){
        
        Class clazz = Class.forName("com.example.basedemo.reflectdemo.Fruit");
        Constructor<Fruit> constructor1 = clazz.getConstructor();
        // 注意 Integer.class表示引用数据类型Integer的class对象,  所以这里使用 int.class 基本数据类型
        Constructor<Fruit> constructor2 = clazz.getConstructor(int.class,String.class,String.class);

        Fruit fruit1 = constructor1.newInstance();
        Fruit fruit2 = constructor2.newInstance(20,"苹果","山东");
       
        System.out.println(fruit1);
        System.out.println(fruit2);
    }

运行结果如下：
Fruit(price=0, name=null, address=null)
Fruit(price=20, name=苹果, address=山东)
```



### 获取Class 中的属性

```java
 public class Fruit {
     public  String check;
     private String checkNumber;
     public Fruit() {}
   }
   
 public static void main(String[] args) {
        
     Class clazz = Class.forName("com.example.basedemo.reflectdemo.Fruit");
     Constructor<Fruit> constructor1 = clazz.getConstructor();
     Fruit fruit1 = constructor1.newInstance();

     // public 属性
     Field publicField =  clazz.getField("check");    
     publicField.set(fruit1, "安检");
     Object publicType = publicField.get(fruit1);
     System.out.println("public 属性赋值-----"+publicType);
     System.out.println(fruit1.check);

     // private 属性 
     Field privateField =  clazz.getDeclaredField("checkNumber");    
     privateField.setAccessible(true);  //对私有字段的访问取消检查
     privateField.set(fruit1, "安检次数");
     Object privateType = privateField.get(fruit1);
     System.out.println("private 属性赋值-----"+privateType);
   }
运行结果：
public 属性赋值-----安检
安检
private 属性赋值-----安检次数
```



### Class 中的方法并运行

```java
 public class Fruit {
    private int price;
    private String name;
    private String address;
    public Fruit() {}
    
    public void show() {
       System.out.println("show 无参数方法被调用");
    }

    public String show(String messsage) {
        return "show 有参数方法被调用: 打印消息--->" + messsage;
    }
}

 public static void main(String[] args){

    Class clazz = Class.forName("com.example.basedemo.reflectdemo.Fruit");
    Constructor<Fruit> constructor1 = clazz.getConstructor();
    Fruit fruit1 = constructor1.newInstance();

    Method method = clazz.getMethod("show", null); // 获取空参数show 方法
    method.invoke(fruit1, null); // 执行无参方法

    Method  method2 = clazz.getMethod("show",String.class); //获取有参show 方法
    String resultMessage = (String) method2.invoke(fruit1, "执行中...."); // 执行有参方法
    System.out.println(resultMessage);
 }

运行结果：
show 无参数方法被调用
show 有参数方法被调用: 打印消息--->执行中....
```



## 常用创建对象通用方法

```java
 public static  <T> T newInstance(Class<T> clazz) {
     String cName = clazz.getName();  
     try {
         return (T) Class.forName(cName).newInstance();  
     } catch (Exception e) {
         throw new RuntimeException(e);
     }
 }
```

# IOC

Spring 中的 IOC 的底层实现原理就是反射机制，Spring 的容器会帮我们创建实例，该容器中使用的方法就是反射，通过解析xml文件，获取到id属性和class属性里面的内容，利用反射原理创建配置文件里类的实例对象，存入到Spring的bean容器中。

# 获取反射的三种方法

1. 通过new对象实现反射机制 

2. 通过路径实现反射机制 

3. 通过类名实现反射机制

```java
 public class Student {
        private int id;
        String name;
        protected boolean sex;
        public float score;
    }
    
     public static void main(String[] args) {
        // 方式一(通过建立对象)
        Student stu = new Student();
        Class classobj1 = stu.getClass();
        System.out.println(classobj1.getName());
        // 方式二（所在通过路径-相对路径）
        Class classobj2 = Class.forName("fanshe.Student");
        System.out.println(classobj2.getName());
        // 方式三（通过类名）
        Class classobj3 = Student.class;
        System.out.println(classobj3.getName());
    }
    
```





# 反射使用注意

说了这么多，那么我们的开发中，为什么不全部都用反射那？一个原因，开销，它的开销是什么昂贵的，随意尽量在最需要的地方使用反射。
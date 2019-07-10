---
title:       "jdk1.8新特性-Optional"
subtitle:    ""
description: ""
date:        2019-07-03
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "Optional", "防止空指针"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://juejin.im/post/5c0b63336fb9a049f43b221b**

# Optional定义

本质上，这是一个包含有**可选值的包装类**，这意味着 Optional 类既可以含有对象也可以为空。

# Optional常用方法

**Optional 主要用作返回类型。**在获取到这个类型的实例后，如果它有值，你可以取得这个值，否则可以进行一些替代行为

*Optional* 类有一个非常有用的用例，就是将其与流或其它返回 *Optional* 的方法结合，以**构建流畅的API**。

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private int id;
    private String userName;
    private String passWord;

    public static void main(String[] args) {
         User  user = new User(1,"麦子","1989");

         // 创建一个Optional实例
         Optional op = Optional.of(user);
         System.out.println(op.get());

         // 创建一个Optional实例
         op = Optional.empty();
         System.out.println(op);
  
         // 若t不为null,创建optional实例，否则创建空实例
        //  user = null;
         op = Optional.ofNullable(user);
         System.out.println(op);

         // 判断是否包含值
         boolean flag = op.isPresent();
         System.out.println(flag);

         // 如果调用对象包含值，返回改值，否则返回传入的对象, 
         // 这个地方可以用再方法的返回值， 避免null指针。
         op = Optional.empty();
         User resultUser = (User) op.orElse(new User(2, "小强", "123"));
         System.out.println(resultUser);


         // 如果调用对象包含值，返回该值， 否则返回 参数 获取的值 
        //  op = Optional.of(new User(5, "5号", "1234567"));
         User user3 = (User) op.orElseGet(() -> new User(3, "无敌", "456"));
         System.out.println(user3);

     
         // 如果有值对其处理，并返回处理后的Optioal,否则返回Optional.empty();
         // 这个地方就可以和stream流来结合处理
         User user6 = new User(6, "6号", "66666");
         String userName = Optional.ofNullable(user6).map(u -> u.getUserName()).orElse("default@gmail.com");
         System.out.println(userName);

         // 过滤数据
         Optional<User>  result = Optional.ofNullable(user6).filter(x -> x.getUserName().equals("7号"));
         System.out.println(result.isPresent());
      
    }

}
```

![Xnip2019-07-02_00-25-25](/img/Xnip2019-07-02_00-25-25.png)

**注意： 除了 *orElse()* 和 *orElseGet()* 方法，Optional 还定义了 *orElseThrow()* API —— 它会在对象为空的时候抛出异常，而不是返回备选的值：**

```java
public void exception() {
    User result = Optional.ofNullable(user)
      .orElseThrow( () -> new IllegalArgumentException());
}
```

**在这里我们就可以直接抛出自己定义的一些业务异常。** 

# Optional想解决什么问题

Optional通过包装一个对象，形成一个更具表现力的容器，用来减少Java系统中空指针异常的数量，Optional的API考虑了返回值丢失为`null`的可能性。

如果Java最开始就有Optional类，那么大多数库和应用程序可能会更好地处理丢失的返回值，减少空指针异常的数量和错误总数。

# Optional不应该被怎样使用？

Optional不适合在下列场景中使用，不会有任何好处：

- 在域模型层（不可序列化）
- 在DTO中（不可序列化）
- 函数的输入参数
- 构造函数的参数
- 避免使用Optional作为类或者实例的属性，而应该在返回值中用来包装返回实例对象。
- 避免使用Optional.get()方式来获取实例对象，因为使用前需要使用Optional.isPresent()来检查实例是否存在，否则会出现NPE问题。
- 避免使用Optional.isPresent()来检查实例是否存在，因为这种方式和null != obj没有区别，这样用就没什么意义了。

# Optional  应该怎样用

实例对象存在则返回，否则提供默认值或者通过方法来设置返回值，即使用orElse/orElseGet方式：












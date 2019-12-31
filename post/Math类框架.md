---
title:       "Math类框架"
subtitle:    ""
description: ""
date:        2019-12-26
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["java基础"]
categories:  ["Tech" ]
---

[TOC]

转载地址：https://juejin.im/post/5d1b7ca9e51d4577407b1d92

# 概述

类中包含基本的数字操作，如指数、对数、平方根和三角函数。java.math是一个包，提供用于执行任意精度整数(BigInteger)算法和任意精度小数(BigDecimal)算法的类。

我们经常用到的数据类型int，short，long，float，double的精度虽然能满足我们日常需要，但是如果遇到高精度计算就会导致数据不准确，所以会使用math包下的任意精度的类操作数据。

# BigInteger

BigInteger有三个静态的常数，分别为ONE, TEM, ZERO

```java
 public static void main(String[] args) {
        System.out.println(BigInteger.ONE);//1
        System.out.println(BigInteger.TEN);//10
        System.out.println(BigInteger.ZERO);//0
    }

```

BigInteger构造函数中有一个将十进制字符串转成成BigInteger,注意：BigInteger没有无参构造函数

```java
BigInteger  bigInteger = new BigInteger("100")//100
```

BigInteger还可以通过valueOf方法将普通数值转成大数值

```java
BigInteger valueOf = BigInteger.valueOf(1000);
System.out.println(valueOf);
```

**注意**：BigInteger不能通过基本数据类型的加减乘除(+,-,*,/)方式处理，而是通过方法来处理



## 常用方法

BigInteger提供了各种数学计算的方式，比如绝对值，异或运算

```java
    public static void main(String[] args) {
        BigInteger bigInteger = new BigInteger("100");
        BigInteger valueOf = BigInteger.valueOf(1000);
        //加法
        BigInteger add = valueOf.add(bigInteger);
        //减法
        BigInteger subtract = valueOf.subtract(bigInteger);
        //除法
        BigInteger divide = valueOf.divide(bigInteger);
        //乘法
        BigInteger multiply = valueOf.multiply(bigInteger);
        //转成double  当然floatValue，intValue，longValue
        double v = valueOf.doubleValue();
        //比较是否相等
        boolean equals = valueOf.equals(bigInteger);
        //求负数
        BigInteger negate = valueOf.negate();
        //转换成字符串
        String s = valueOf.toString();
        BigInteger xor = valueOf.xor(bigInteger);
        System.out.println(xor);//908
        System.out.println(s);//1000
        System.out.println(negate);//-1000
        System.out.println(equals);//false
        System.out.println(v);//1000.0
        System.out.println(multiply);//100000
        System.out.println(divide);//10
        System.out.println(subtract);//900
        System.out.println(add);//1100
    }
```

# BigDecimal

float和double类型的主要设计目标是为了科学计算和工程计算。他们执行二进制浮点运算，这是为了在广域数值范围上提供较为精确的快速近似计算而精心设计的。然而，它们没有提供完全精确的结果，所以不应该被用于要求精确结果的场合。但是，商业计算往往要求结果精确，这时候BigDecimal就派上大用场啦。	

```java
    public static void main(String[] args) {
        System.out.println(0.2 + 0.1);
        System.out.println(0.3 - 0.1);	
        System.out.println(0.2 * 0.1);
        System.out.println(0.3 / 0.1);
    }

运行结果：
0.30000000000000004
0.19999999999999998
0.020000000000000004
2.9999999999999996

```

你认为你看错了，但结果却是是这样的。问题在哪里呢？原因在于我们的计算机是二进制的。浮点数没有办法是用二进制进行精确表示。我们的CPU表示浮点数由两个部分组成：**指数和尾数**，这样的表示方法一般都会失去一定的精确度，有些浮点数运算也会产生一定的误差。如：2.4的二进制表示并非就是精确的2.4。反而最为接近的二进制表示是 2.3999999999999999。浮点数的值实际上是由一个特定的数学公式计算得到的。

## 构造方法

1. public BigDecimal(double val) 将double表示形式转换为BigDecimal ***不建议使用**
2. public BigDecimal(int val)  将int表示形式转换成BigDecimal
3. public BigDecimal(String val)  将String表示形式转换成BigDecimal

```java
    public static void main(String[] args)
    {
        BigDecimal bigDecimal = new BigDecimal(2);
        BigDecimal bDouble = new BigDecimal(2.3);
        BigDecimal bString = new BigDecimal("2.3");
        System.out.println("bigDecimal=" + bigDecimal);
        System.out.println("bDouble=" + bDouble);
        System.out.println("bString=" + bString);
    }

运行结果：

bigDecimal=2
bDouble=2.29999999999999982236431605997495353221893310546875
bString=2.3
```

**为什么会出现这种情况呢？**

JDK的描述：参数类型为double的构造方法的结果有一定的不可预知性。有人可能认为在Java中写入newBigDecimal(0.1)所创建的BigDecimal正好等于 0.1（非标度值 1，其标度为 1），但是它实际上等于0.1000000000000000055511151231257827021181583404541015625。这是因为0.1无法准确地表示为 double（或者说对于该情况，不能表示为任何有限长度的二进制小数）。这样，传入到构造方法的值不会正好等于 0.1（虽然表面上等于该值）。

另一方面，String 构造方法是完全可预知的：写入 newBigDecimal("0.1") 将创建一个 BigDecimal，它正好等于预期的 0.1。因此，比较而言，通常建议优先使用String构造方法。

**当double必须用作BigDecimal时，请使用Double.toString(double)转成String，然后使用String构造方法，或使用BigDecimal的静态方法valueOf，如下** 

```java
    public static void main(String[] args)
    {
        BigDecimal bDouble1 = BigDecimal.valueOf(2.3);
        BigDecimal bDouble2 = new BigDecimal(Double.toString(2.3));

        System.out.println("bDouble1=" + bDouble1);
        System.out.println("bDouble2=" + bDouble2);

    }

运算结果：
bDouble1=2.3
bDouble2=2.3
```



## BigDecimal运算

对于常用的加，减，乘，除，BigDecimal类提供了相应的成员方法。

```java
public BigDecimal add(BigDecimal value);                        //加法

public BigDecimal subtract(BigDecimal value);                   //减法 

public BigDecimal multiply(BigDecimal value);                   //乘法

public BigDecimal divide(BigDecimal value);                     //除法

```

列如：

```java
public static void main(String[] args)
    {
        BigDecimal a = new BigDecimal("4.5");
        BigDecimal b = new BigDecimal("1.5");

        System.out.println("a + b =" + a.add(b));
        System.out.println("a - b =" + a.subtract(b));
        System.out.println("a * b =" + a.multiply(b));
        System.out.println("a / b =" + a.divide(b));
    }

结果：
a + b =6.0
a - b =3.0
a * b =6.75
a / b =3
```

这里有一点需要**注意的是除法运算divide**.

BigDecimal除法可能出现不能整除的情况，比如 4.5/1.3，这时会报错java.lang.ArithmeticException: Non-terminating decimal expansion; no exact representable decimal result.

其实divide方法有可以传三个参数

public BigDecimal divide(BigDecimal divisor, int scale, int roundingMode) 第一参数表示除数， 第二个参数表示小数点后保留位数， 第三个参数表示舍入模式，只有在作除法运算或四舍五入时才用到舍入模式，有下面这几种

```java
ROUND_CEILING    //向正无穷方向舍入

ROUND_DOWN    //向零方向舍入

ROUND_FLOOR    //向负无穷方向舍入

ROUND_HALF_DOWN    //向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，向下舍入, 例如1.55 保留一位小数结果为1.5

ROUND_HALF_EVEN    //向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，如果保留位数是奇数，使用ROUND_HALF_UP，如果是偶数，使用ROUND_HALF_DOWN

ROUND_HALF_UP    //向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，向上舍入, 1.55保留一位小数结果为1.6

ROUND_UNNECESSARY    //计算结果是精确的，不需要舍入模式

ROUND_UP    //向远离0的方向舍入

```

按照各自的需要，可传入合适的第三个参数。四舍五入采用 ROUND_HALF_UP

需要对BigDecimal进行截断和四舍五入可用setScale方法，例：

```java
public static void main(String[] args)
    {
        BigDecimal a = new BigDecimal("4.5635");

        a = a.setScale(3, RoundingMode.HALF_UP);    //保留3位小数，且四舍五入
        System.out.println(a);//4.564
    }

```

减乘除其实最终都返回的是一个新的BigDecimal对象，因为BigInteger与BigDecimal都是不可变的（immutable）的，在进行每一步运算时，都会产生一个新的对象。

```java
    public static void main(String[] args)
    {
        BigDecimal a = new BigDecimal("4.5");
        BigDecimal b = new BigDecimal("1.5");
        BigDecimal add = a.add(b);

        System.out.println(add);//6.0
        System.out.println(a);  //输出4.5. 加减乘除方法会返回一个新的BigDecimal对象，原来的a不变
    }

```



## 注意

1. 商业计算使用BigDecimal。
2. 尽量使用参数类型为String的构造函数。 
3.  BigDecimal都是不可变的（immutable）的，在进行每一步运算时，都会产生一个新的对象，所以在做加减乘除运算时千万要保存操作后的值。
4. 我们往往容易忽略JDK底层的一些实现细节，导致出现错误，需要多加注意。



# 静态方法

转载地址：https://www.cnblogs.com/zhangmingcheng/p/4338246.html

类中包含E和PI两个静态常量，以及进行科学计算的类(static)方法，可以直接通过类名调用。

```java
public static final Double E = 2.7182818284590452354 

public static final Double PI = 3.14159265358979323846 

public static long abs(double x)：传回x 的绝对值。 X也可int long float 

public static long sin(double x): 传回x径度的正弦函数值

public static long cos(double x)：传回x径度的余弦函数值

public static long tan(double x): 传回x径度的正切函数值

public static long asin(double x)：传回x值的反正弦函数值。

public static long acos(double x)：传回x值的反余弦函数值。

public static long atan(double x)：传回x值的反正切函数值。

public static long atan2(double x, double y)：传回极坐标（polar）的θ值

public static long floor(double x)：传回不大于x的最大整数值

public static long ceil(double x)：传回不小于x的最小整数值。

public static long exp(double x)：传回相当于e^x值

public static long log(double x)：传回x的自然对数函数值

public static long max(double x,double y)：传回x、y较大数

public static long min(double x,double y)：传回x、y较小数

public static long pow(double x,double y)：传回x的y次幂值

public static long sqrt(double x): 传回x开平方值

public static long rint(double x):传回最接近x的整数值

public static long round(double x):传回x的四舍五入值

public static long toDegrees(double angrad):传回将angrad径度转换成角度

public static long toRadians(double angdeg): 传回将angdeg角度转换成径度

public static long random():传回随机数值,产生一个0-1之间的随机数(不包括0和1) 

Math.log(100)/Math.log(10); 表示的是100的以10为底的对数函数的值，是2

向上取整用Math.ceil(double a) 
向下取整用Math.floor(double a) 
```


---
title:       "java中的标号:outer的作用"
subtitle:    ""
description: ""
date:        2020-03-07
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "java中的标号:outer的作用"]
categories:  ["Tech" ]
---

转载地址：https://blog.csdn.net/m0_37468234/article/details/78937892?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task

# 概述

标号提供了一种简单的break语句所不能实现的控制循环的方法，当在循环语句中遇到break时，不管其它控制变量，都会终止。outer用于解决当嵌套在几层循环中想退出循环时的情况。正常的break只退出一重循环，你可以用标号标出你想退出哪一个语句。

# 例子1

```java
public class testo {
	public static void main(String args[]) {
		outer:
		for(int i=0; i<10;i++) {
			
			for(int j=0;j<10;j++) {
				System.out.println(i+" "+j);
				if(i==1) {
					break outer;
				}
			}
		}
		
	}
}

输出结果：
0 0
0 1
0 2
0 3
0 4
0 5
0 6
0 7
0 8
0 9
1 0
```

加了outer 之后可以在满足条件之时，直接调到outer所在的区域去；

# 例子2

```java
public class testo {
	public static void main(String args[]) {
		
		for(int i=0; i<5;i++) {
			outer:
			for(int j=0;j<10;j++) {
				System.out.println(i+" "+j);
				if(j==2) {
					break outer;
				}
			}
		}
		
	}
}

结果为：
0 0
0 1
0 2
1 0
1 1
1 2
2 0
2 1
2 2
3 0
3 1
3 2
4 0
4 1
4 2
```

# 总结

为了解决， 多层次的循环的时候， 退出哪一层的Flag标志。 


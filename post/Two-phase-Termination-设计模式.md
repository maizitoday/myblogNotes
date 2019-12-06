---
title:       "Two-phase-Termination-设计模式"
subtitle:    ""
description: ""
date:        2019-12-06
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["多线程和高并发", "两阶段终止模式", "设计模式"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://blog.csdn.net/crazyzxljing0621/article/details/56669649**

# 概述

当我们想要结束一个线程或者关闭jvm的时候，通过此模式可以优雅安全的关闭线程，让线程可以完成它本应完成的当前任务并可以附加一些收尾工作后再进行关闭。 此模式下关闭线程会有一定延迟，主要在于被关闭线程需要执行完后，再进行关闭。

# 代码演示

## IGameSys--游戏系统抽象模板

定义了系统执行周期，系统名，规范了event和overevent，IgameSys及其派生类无需关心Thread细节，只需完成event/overevent业务操作即可。

```java
/**
 * 游戏系统
 * @author Administrator
 *
 */
public abstract class IGameSys {
	private long time;
	private String name;
 
	public IGameSys(long time, String name) {
		this.time = time;
		this.name = name;
	}
 
 
	public String name() {
		return this.name;
	}
 
	public long time() {
		return this.time;
	}
 
	public abstract void event(Object... ob) throws ThreadOverException;
 
	public abstract void overEvent();
 
}
```

## ActivitySys--活动系统实现

```java
 /**
 * 活动系统实现
 * 
 * @author Allen
 * @date 2017年2月21日
 * 
 */
public class ActivitySys extends IGameSys {
 
	public ActivitySys(long time, String name) {
		super(time, name);
	}
 
	private boolean temp = false;
 
	@Override
	public void event(Object... ob) throws ThreadOverException {
		if (!temp){
			System.out.println("<Thread-" + super.name() + ">[event]活动系统运行中 ><");
			temp=true; 
		}
	}
 
	@Override
	public void overEvent() {
		System.out.println("<Thread" + super.name() + ">[overEvent]");
	}
 
}
```

## MsgSys--公告系统实现

```java
 /**
 * 公告系统实现
 * 
 * @author Allen
 * @date 2017年2月21日
 * 
 */
public class MsgSys extends IGameSys {
 
	public MsgSys(long time, String name) {
		super(time, name);
	}
	private boolean temp=false;
	@Override
	public void event(Object... ob) throws ThreadOverException {
		if (!temp){
		System.out.println("<Thread-" + super.name() + ">[event]系统公告运行中 ><");
		temp=true;
		}
	}
 
	@Override
	public void overEvent() {
		System.out.println("<Thread" + super.name() + ">[overEvent]");
	}
 
}


```

## IRepairModel

**日后扩展，内容规范，依赖规范。这个在常用设计时候，应该这样的去考虑设计。**

```java
public interface IRepairModel extends Runnable {
 
}
```



## RepairModelImpl   两阶段终止模式实现

对IgameSys进行了两阶段终止模式实现。

```java

public class RepairModelImpl implements IRepairModel {
 
	private IGameSys ir;
 
	public RepairModelImpl(IGameSys ir) {
		this.ir = ir;
	}
 
	@Override
	public void run() {
		System.out.println(ir.name() + "已启动");
		try {
			while (true) {
				Thread.sleep(ir.time());
				if (SystemState.getInstance().isState()){
            break;
         }	
				ir.event();
			}
		} catch (ThreadOverException e) {
			e.printStackTrace();
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			ir.overEvent();
		}
		return;
	}
}
```



## SystemState

单例且volatile，sendState开闭。

```java
 /**
 * 修复状态
 * 
 * @author Allen
 * @date 2017年2月21日
 *
 */
public final class SystemState {
	/**
	 * 饿汉单例
	 */
	private static SystemState instance = new SystemState();
 
	private SystemState() {
	}
 
	public static SystemState getInstance() {
		return instance;
	}
 
	private volatile boolean state = false;
 
	public boolean isState() {
		return state;
	}
 
	/**
	 * 发送状态 
	 * @author Allen
	 * @date 2017年2月21日
	 */
	public void sendState() {
		this.state = true;
 	}

}
```

## ThreadOverException

```java
 public class ThreadOverException extends Exception {
 
	private static final long serialVersionUID = 1L;
 
}


```



## Main 

```java
/**
 * 
 * @author Allen 
 * @date 2017年2月22日
 *
 */
public class Main {
	static IGameSys[] igame = { 
		new MsgSys(500,"公告系统"), 
		new ActivitySys(200,"活动系统")
		}; 
	public static void main(String[] args) throws InterruptedException {
		System.out.println("游戏服务器已启动");
		startThreadGroup();
		Thread.sleep(4000);
		SystemState.getInstance().sendState();
 
	}
	private static void startThreadGroup() throws InterruptedException { 
		Iterator<IGameSys> it = Arrays.asList(igame).iterator();
			while (it.hasNext()) {
				Thread thread = new Thread(new RepairModelImpl(it.next()));
				thread.start(); 
			} 
	}
}


```



## 运行结果

```

```



# 总结

此模式的设计其实没什么精髓之处，其实就是线程中不断对一个参数进行验证，满足条件时线程跳出并执行结束后的收尾工作，最后安全关闭。

# 疑惑

感觉就是一个线程结束后， 然后在finally里面或者其他的地方做一些结尾的工作处理而已。
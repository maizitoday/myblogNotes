---
title:       "private static final long serialVersionUID = 1L"
subtitle:    ""
description: ""
date:        2020-03-01
author:      "麦子"
image:       "https://c.pxhere.com/images/56/96/af639ed6d305615b47ff1d89804b-1423003.jpg!d"
tags:        ["java相关概念", "serialVersionUID=1L"]
categories:  ["Tech" ]
---

转载地址：https://www.jianshu.com/p/5ec98a11fe63

# 概述说明

```java
import java.io.Serializable;

public class User implements Serializable {

    /**
     *
     */
    private static final long serialVersionUID = 1L;
    
}
```

serialVersionUID作用：**相当于java类的身份证。主要用于版本控制**。
serialVersionUID作用是序列化时保持版本的兼容性，即在版本升级时反序列化仍保持对象的唯一性。

# 有两种生成方式

 一个是默认的1L

```java
比如：private static final long serialVersionUID = 1L; 
```


 一个是根据类名、接口名、成员方法及属性等来生成一个64位的哈希字段

```java
比如：private static final long serialVersionUID = xxxxL;
```

# 说明

如果你没有考虑到兼容性问题时，就把它关掉，不过有这个功能是好的，只要任何类别实现了Serializable这个接口的话，如果没有加入serialVersionUID，Eclipse都会给你warning提示，这个serialVersionUID为了让该类别Serializable[向后兼容](https://www.baidu.com/s?wd=向后兼容&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)。

如果你的类Serialized存到硬盘上面后，可是后来你却更改了类别的field(增加或减少或改名)，当你Deserialize时，就会出现Exception的，这样就会造成不兼容性的问题。

但当serialVersionUID相同时，它就会将不一样的field以type的预设值Deserialize，可避开不兼容性问题。
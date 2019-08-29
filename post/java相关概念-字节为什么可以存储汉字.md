---
title:       "字节为什么可以存储汉字"
subtitle:    ""
description: ""
date:        2019-08-29
author:      "麦子"
image:       "https://c.pxhere.com/images/56/96/af639ed6d305615b47ff1d89804b-1423003.jpg!d"
tags:        ["java相关概念", "字节存储汉字"]
categories:  ["Tech" ]
---

[TOC]

转载地址：<https://www.zhihu.com/question/23374078>   [邱昊宇](https://www.zhihu.com/people/timothyqiu)

转载地址：<https://blog.csdn.net/happyaaaaaaaaaaa/article/details/50978167>

# Unicode 是「字符集」

Unicode（统一码、万国码、单一码）是一种在计算机上使用的字符编码。**它为每种语言中的每个字符设定了统一并且唯一的二进制编码，以满足跨语言、跨平台进行文本转换、处理的要求。**

为每一个「字符」分配一个唯一的 ID（学名为码位 / 码点 / Code Point）， 

**广义的 Unicode 是一个标准**，定义了一个字符集以及一系列的编码规则，即 Unicode 字符集和 UTF-8、UTF-16、UTF-32 等等编码……

Unicode 字符集为每一个字符分配一个码位，例如「知」的码位是 30693，记作 U+77E5（30693 的十六进制为 0x77E5）。

# UTF-8 是「编码规则」

将「码位」转换为字节序列的规则（编码/解码 **可以理解为 加密/解密 的过程**）



# char存储汉字

char是按照字符存储的，不管英文还是中文，固定占用占用2个字节，用来储存Unicode字符。范围在0-65535。
unicode编码字符集中包含了汉字，所以，char型变量中当然可以存储汉字啦。不过，如果某个特殊的汉字没有
被包含在unicode编码字符集中，那么，这个char型变量中就不能存储这个特殊汉字。**目前的用于实用的 Unicode 版本对应于 UCS-2，使用16位的编码空间。也就是每个字符占用2个字节。**



# 不同的看编码占据字节数也不同

## 1.  utf-32中文是4字节；

## 2.  utf-8码的中文都是3字节的，字母是1字节，因为utf-8是变长编码；

## 3.  gbk/gbk18030 中文是2字节的，英文是1个字节。


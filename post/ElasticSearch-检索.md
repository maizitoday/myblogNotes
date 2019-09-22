---
title:       "ElasticSearch-检索原理"
subtitle:    ""
description: ""
date:        2019-09-22
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["elasticSearch","检索原理","分词器","ik插件","自定义分词器"]
categories:  ["Tech" ]
---

[TOC]

**说明：本文来自于https://www.bilibili.com/video/av64033816/?p=1>视频讲解**

**官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/6.0/getting-started.html**

# 检索

```json
#查
GET my_inde/doc/1

#全部查询
GET my_inde/doc/_search

#全文检索 ， 只要某一个字段包含就可以查询出来
GET my_inde/doc/_search?q=180

#模糊检索 ， 只要某一个字段包含就可以查询出来
GET my_inde/doc/_search?q=yb
```

# 搜索原理

![Xnip2019-09-22_16-24-24](/img/Xnip2019-09-22_16-24-24.png)

如上图演示的百度搜索引擎处理方式。 

1. 通过爬虫进行填充内部的数据仓库
2. 通过对文档进行分词，根据title分出不同的词语出来。 
3. 然后进行一个倒排索引处理，也就是根据你分出来的词语做一个统计，把一个词语对应的索引都总结出来。
4. 当你在搜索框输入词语的时候， 他会根据和倒排索引的对应，快速的找到对应的文档数据出来。 

**注意**：如上图可以看出来， 其中倒排索引是最影响效率的。 



## B+Tree数据算法

倒排索引

![Xnip2019-09-22_16-38-12](/img/Xnip2019-09-22_16-38-12.png)

具体演示地址：https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html

其中每一个倒排索引中，都会有一个Posting List， 这里面包含了文档的id。 如图就是下面的红色框。

![Xnip2019-09-22_16-40-29](/img/Xnip2019-09-22_16-40-29.png)



## Posting List

| DocId |  TF  | Position | offset |
| :---: | :--: | :------: | :----: |
|   1   |  1   |    0     | <0,2>  |
|   3   |  1   |    0     | <0,2>  |

**DocId**

文档id，文档的原始信息

**TF**

单词频率，记录该词在文档中出现的次数，用于后续相关性算法

**Position**

位置，记录Fidld分词后，单词所在的位置，从0开始

**offset**

偏移量，记录单词在文档中开始和结束位置，用于高亮显示等。



## 注意

每个文档字段都有自己的倒排索引。 



# 分词器

|       步骤        |            处理            | 应用场景                                 |
| :---------------: | :------------------------: | ---------------------------------------- |
| Character  Filter |   对**原始文本**进行处理   | 去除html标签,特殊字符                    |
|     Tokenizer     |   将**原始文本**进行分词   | 如：培训机构->培训,机构                  |
|   Token Filters   | 分词后的**关键字**进行加工 | 如：转小写，删除语气词，近义词和同义词等 |

![Xnip2019-09-22_19-42-07](/img/Xnip2019-09-22_19-42-07.png)

## 演示效果

```json
POST _analyze
{
  "analyzer": "standard",
  "text": "hello world"
}


结果
{
  "tokens" : [
    {
      "token" : "hello",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "world",
      "start_offset" : 6,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 1
    }
  ]
}

```

如上可以看到， 他对hello world 进行分词处理。



## 针对索引字段进行分词

可以用下面的进行自己文档的分词效果。

```json
POST my_inde/_analyze
{
  "field": "name",
  "text": "are you ok"
}


```



## 内置分词器

Elasticsearch有许多内置的分词器，可用于构建 [自定义分词器](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/analysis-custom-analyzer.html)。

### 面向字断词

以下分词器通常用于将全文标记为单个单词

|           名称            | 关键字        |
| :-----------------------: | :------------ |
|        标准分词器         | standard      |
|        字母分词器         | letter        |
|        小写分词器         | lowercase     |
|        空格分词器         | whitespace    |
| UAX URL电子邮件令牌生成器 | uax_url_email |
|        经典分词器         | classic       |
|        泰语分词器         | thai          |
|        停止分词器         | stop          |

1. **标准分词器**

该`standard`标记生成器将文本分为单词边界条件，由Unicode文本分割算法的定义。它删除大多数标点符号。这是大多数语言的最佳选择。

2. **字母分词器**

该`letter`标记生成器将文本分为方面每当遇到一个字符是不是字母

3. **小写分词器**

该`lowercase`标记生成器，如`letter`标记生成器，文本分为方面每当遇到一个特点，它已不是一个字母，但它也小写的所有条款。

4. **空格分词器**

该`whitespace`标记生成器将文本分为方面每当遇到任何空白字符。

5. **UAX URL电子邮件令牌分词器**

该`uax_url_email`分词器是一样的`standard`，除了它识别URL和电子邮件地址作为单个记号标记生成器。

6. **经典分词器**

该`classic`分词器是针对英语语法基础分词器。

7. **泰语分词器**

分`thai`词器将泰语文本分割为单词。



### 部分字断词

这些分词器将文本或单词分成小片段，以实现部分单词匹配：

|         名称          | 关键字     |
| :-------------------: | :--------- |
|   N-Gram令牌分词器    | ngram      |
| Edge N-Gram令牌分词器 | edge_ngram |

1. **N-Gram令牌分词器**

所述`ngram`标记生成器可以分解文本成单词，当它遇到任何指定的字符的列表（例如，空格或标点），则它返回的n-gram的每个单词的：连续字母的滑动窗口，例如`quick`→ `[qu, ui, ic, ck]`。

2. **Edge N-Gram令牌分词器**

所述`edge_ngram`标记生成器可以分解文本成单词，当它遇到任何指定的字符的列表（例如，空格或标点），则它返回的n-gram，其锚定到这个词，例如开始每个单词的`quick`→ `[q, qu, qui, quic, quick]`。



### 结构化文本断词

以下分词器通常与标识符，电子邮件地址，邮政编码和路径之类的结构化文本一起使用，而不是全文本：

|        名称        | 关键字               |
| :----------------: | :------------------- |
|    关键字分词器    | keyword              |
|     模式分词器     | pattern              |
|   简单模式分词器   | simple_pattern       |
|    字符组分词器    | char_group           |
| 简单模式分割分词器 | simple_pattern_split |
|     路径分词器     | path_hierarchy       |

1. **关键字分词器**

   该`keyword`分词器是一个“ 空操作 ”标记生成器接受任何文本它被赋予并输出完全相同的文本作为一个单项。可以将其与令牌过滤器结合使用，[`lowercase`](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/analysis-lowercase-tokenfilter.html)以对分析的术语进行标准化。

2. **模式分词器**

   所述`pattern`标记生成器使用正则表达式要么分裂文本术语每当一个字分离器相匹配，或者捕捉到的匹配的文本作为术语。

3. **简单模式分词器**

   分`simple_pattern`词器使用正则表达式将匹配的文本捕获为术语。它使用正则表达式功能的受限子集，并且通常`pattern`比分词器更快。

4. **字符组分词器**

   所述`char_group`标记生成器是可配置的通过字符集合分裂上，这通常是比运行正则表达式更便宜。

5. **简单模式分割分词器**

   所述`simple_pattern_split`标记生成器使用相同的受限的正则表达式的子集作为`simple_pattern`标记生成器，但在分割匹配的输入，而不是返回的匹配，而术语。

6. **路径分词器**

   所述`path_hierarchy`标记生成器需要像文件系统路径的分层值，分割的路径分隔，并发出一个术语，树中的每个部件，例如`/foo/bar/baz`→ `[/foo, /foo/bar, /foo/bar/baz ]`。



# 自定义分词器

### 格式

`custom`分析器接受以下参数：

|          参数          | 说明                                                         |
| :--------------------: | :----------------------------------------------------------- |
|       tokenizer        | 内置或自定义的[分词器](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/analysis-tokenizers.html) |
|      char_filter       | 内置或自定义[字符过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/analysis-charfilters.html)的**可选数组** |
|         filter         | 内置或自定义[令牌过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/analysis-tokenfilters.html)的**可选数组** |
| position_increment_gap | 在为文本值数组建立索引时，Elasticsearch在一个值的最后一项和下一个值的第一项之间插入一个假的“空白”，以确保词组查询与来自不同数组元素的两项不匹配。默认为`100`。查看[`position_increment_gap`](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/position-increment-gap.html)更多。 |

### 示例

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {   
          "type":      "custom",  
          "tokenizer": "standard",// 第二步：分词
          "char_filter": [ // 第一步： 字符过滤
            "html_strip"
          ],
          "filter": [ // 第三步：关键字加工处理
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  }
  
#对照上面的分词流程图可以看到就是执行上面的三个步骤。 
  
POST my_index/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "Is this <b>déjà vu</b>?"
}



{
  "tokens" : [
    {
      "token" : "is",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "this",
      "start_offset" : 3,
      "end_offset" : 7,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "deja",
      "start_offset" : 11,
      "end_offset" : 15,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "vu",
      "start_offset" : 16,
      "end_offset" : 22,
      "type" : "<ALPHANUM>",
      "position" : 3
    }
  ]
}
```



# ik插件安装

https://github.com/medcl/elasticsearch-analysis-ik/releases/ 找到对应的版本下载， 放入到容器对应的目录中。

```dockerfile
docker cp 
/elasticsearch-analysis-ik-6.8.3.zip c350e8e2732c:/usr/share/elasticsearch/plugins/ik
```

ik 带有两个分词器
**ik_max_word ：**会将文本做最细粒度的拆分；尽可能多的拆分出词语
**ik_smart：**会做最粗粒度的拆分；已被分出的词语将不会再次被其它词语占有

效果如下：

```json
POST _analyze
{
  "tokenizer": "ik_smart"
  , "text": ["php是世界上最好的语言"]
  
}


{
  "tokens" : [
    {
      "token" : "php",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "ENGLISH",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "世界上",
      "start_offset" : 4,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "最好",
      "start_offset" : 7,
      "end_offset" : 9,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "的",
      "start_offset" : 9,
      "end_offset" : 10,
      "type" : "CN_CHAR",
      "position" : 4
    },
    {
      "token" : "语言",
      "start_offset" : 10,
      "end_offset" : 12,
      "type" : "CN_WORD",
      "position" : 5
    }
  ]
}
```


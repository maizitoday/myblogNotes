---
title:       "mongodb-GridFS分布式文件系统"
subtitle:    ""
description: ""
date:        2019-09-16
author:      "麦子"
image:       "https://c.pxhere.com/images/a1/23/147c2090573fcd3fbc38e7b7b5a7-1593571.jpg!d"
tags:        ["mongodb","分布式文件系统"]
categories:  ["Tech" ]
---

[TOC]

**说明：本文来自于<https://www.bilibili.com/video/av36394924/?p=2>视频讲解**

**官方文档：<https://docs.mongodb.com/>  4.2版本**

# 简介

[GridFS](https://docs.mongodb.com/manual/reference/glossary/#term-gridfs)是用于存储和检索**超过**[ BSON -](https://docs.mongodb.com/manual/reference/glossary/#term-bson)文档[大小限制](https://docs.mongodb.com/manual/reference/limits/#limit-bson-document-size)为16 MB的文件的规范。

GridFS不是将文件存储在单个文档中，而是将文件分成多个部分或块[[1\]](https://docs.mongodb.com/manual/core/gridfs/#chunk-disambiguation)，并将每个块存储为单独的文档。默认情况下，GridFS使用默认的块大小255 kB; 也就是说，**GridFS将文件分成255 kB的块**，但最后一个块除外。最后一个块只有必要的大小。类似地，不大于块大小的文件只有最终块，只使用所需的空间和一些额外的元数据。

1.  多机器存储， 备份。 
2. 可以突破一般的文件系统对files的限制
3. 分段存储，不像普通的file system整个存储。 

# mongofiles

```sql
mongofiles <options> <commands> <filename>


--host <hostname><:port>
```



## 命令

```
list <prefix>
列出GridFS存储中的文件。list（例如<prefix>）之后指定的字符 可选地将返回项的列表限制为以该字符串开头的文件。

search <string>
列出GridFS存储中的文件，其名称与任何部分匹配<string>。

put <filename>
将指定文件从本地文件系统复制到GridFS存储中。
 
get <filename>
将指定文件从GridFS存储复制到本地文件系统。
 
```



## 实例操作

把一个pdf文件放入到GridFS中

```shell
root@cdcbbd0f125a:/usr/bin# ./mongofiles -d records put /macfile/110.pdf 
2019-09-16T10:37:02.086+0000    connected to: mongodb://localhost/
2019-09-16T10:37:02.231+0000    added gridFile: /macfile/110.pdf

#不需要创建数据库，他会自动加载。  -d 后面是数据库
```

当导入file的时候， grid会在指定的db上面生成两个collection

### chunks

就是把文件分割成了一个一个的255 kB的chunks块

![Xnip2019-09-16_18-44-40](/img/Xnip2019-09-16_18-44-40.png)

```sql
#查询出当前的一个端的数据。
db.getCollection("fs.chunks").find({n:2})
```



### files

就是文件的相关信息。

![Xnip2019-09-16_18-45-38](/img/Xnip2019-09-16_18-45-38.png)



# 分布式文件系统建立

我们可以通过GridFS加上mongdb的分片，建立一个分布式的文件系统。 

 

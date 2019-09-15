---
title:       "mongodb系列-标准sql和mongodb对比"
subtitle:    ""
description: ""
date:        2019-09-15
author:      "麦子"
image:       "https://c.pxhere.com/images/a1/23/147c2090573fcd3fbc38e7b7b5a7-1593571.jpg!d"
tags:        ["mongodb系列", "SQL术语VSMongoDB术语", "创建和修改表", "CRUD"]
categories:  ["Tech" ]
---

[TOC]

**说明：本文来自于<https://www.bilibili.com/video/av36394924/?p=2>视频讲解**

**官方文档：<https://docs.mongodb.com/>  4.2版本**





# SQL术语VSMongoDB术语

![Xnip2019-09-15_16-58-55](/img/Xnip2019-09-15_16-58-55.png)



# 创建和修改表

![Xnip2019-09-15_17-01-46](/img/Xnip2019-09-15_17-01-46.png)

# CRUD

## 新增一条记录

```sql
INSERT INTO people(user_id,
                  age,
                  status)
VALUES ("bcd001",
        45,
        "A")
        
        
db.people.insertOne(
   { user_id: "bcd001", age: 45, status: "A" }
)        
        
```



## 查询

![Xnip2019-09-15_16-55-51](/img/Xnip2019-09-15_16-55-51.png)

## 修改

![Xnip2019-09-15_17-02-38](/img/Xnip2019-09-15_17-02-38.png)

## 删除

![Xnip2019-09-15_17-03-02](/img/Xnip2019-09-15_17-03-02.png)
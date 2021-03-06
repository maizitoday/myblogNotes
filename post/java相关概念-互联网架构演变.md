---
title:       "互联网架构演变"
subtitle:    ""
description: "QPS、TPS, 从单机到集群架构演变，读写分离和分库分表区别"
date:        2020-03-13
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java相关概念", "互联网架构演变"]
categories:  ["Tech" ]
---

[TOC]

**说明来源一份PPT**



# 什么是QPS、TPS？

QPS：Queries Per Second意思是“每秒查询率”，是一台服务器每秒能够相应的查询次数，是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准

TPS：是TransactionsPerSecond的缩写，也就是事务数/秒。它是软件测试结果的测量单位。一个事务是指一个客户机向服务器发送请求然后服务器做出反应的过程。客户机在发送请时开始计时，收到服务器响应后结束计时，以此来计算使用的时间和完成的事务个数。

# 阶段1：单机集中构建网站

 ![Xnip2020-03-13_10-28-59](/img/Xnip2020-03-13_10-28-59.png)

# 阶段2：Nginx+应用服务器配置集群

![Xnip2020-03-13_10-29-58](/img/Xnip2020-03-13_10-29-58.png)

![Xnip2020-03-13_10-30-48](/img/Xnip2020-03-13_10-30-48.png)

# 阶段3：负载均衡服务器配置集群

![Xnip2020-03-13_10-31-38](/img/Xnip2020-03-13_10-31-38.png)

# 阶段4：CDN+Varnish服务器配置集群

![Xnip2020-03-13_10-32-36](/img/Xnip2020-03-13_10-32-36.png)

# 阶段5：数据库读写分离

![Xnip2020-03-13_10-33-06](/img/Xnip2020-03-13_10-33-06.png)



# 阶段6：NOSQL+分布式搜索引擎

![Xnip2020-03-13_10-33-45](/img/Xnip2020-03-13_10-33-45.png)

# 阶段7：NOSQL(HA) + 分库分表+Mycat

![Xnip2020-03-13_10-34-51](/img/Xnip2020-03-13_10-34-51.png)

# 阶段8：分布式文件系统

![Xnip2020-03-13_10-35-24](/img/Xnip2020-03-13_10-35-24.png)

# 京东2017618交易平台目前的架构体系

![Xnip2020-03-13_10-36-23](/img/Xnip2020-03-13_10-36-23.png)

# 阶段9：应用服务化拆分+消息中间件

![Xnip2020-03-13_10-37-01](/img/Xnip2020-03-13_10-37-01.png)

# 阶段10：微服务架构

![Xnip2020-03-13_10-37-44](/img/Xnip2020-03-13_10-37-44.png)
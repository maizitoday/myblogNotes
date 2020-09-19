---
title:       "mysql系列-常用方法和伪造百万数据函数"
subtitle:    ""
description: "伪造百万数据函数，常用函数"
date:        2020-09-19
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["mysql系列","伪造百万数据函数，常用函数"]
categories:  ["Tech" ]
---

[TOC]

# 伪造百万数据函数

转载地址：<https://blog.csdn.net/jack__frost/article/details/72904318**>



## myisam引擎

```sql
create table testMyIsam(  
id int unsigned primary key auto_increment,  
name varchar(20) not null  
)engine=myisam;  


drop procedure if exists ptestmyisam;
delimiter $$
create procedure ptestmyisam()
begin
declare pid int ;
set pid = 1000000;
while pid>0 
do
insert into testMyIsam(name) values(concat("fuzhu", pid));
set pid = pid-1;
end while;
end $$
 
call ptestmyisam();
```



## innodb引擎

```sql
create table testInnoDB( 
 id int unsigned primary key auto_increment, 
 name varchar(20) not null 
 )engine=innodb;  

 
create procedure ptestInndb()
begin
declare pid int ;
set pid = 1000000;
while pid>0 
do
insert into testInnoDB(name) values(concat("fuzhu", pid));
set pid = pid-1;
end while;
end $$

 
call ptestInndb();

```

# 常用函数



原理的博客中就有常用函数， 把他找出来， 然后统一总结起来。 



[CAST与CONVERT 函数的用法](https://www.cnblogs.com/chenqionghe/p/4675844.html)   



 find_in_set    



 GROUP_CONCAT  分组连接   https://blog.csdn.net/qq_35067322/article/details/104218222  

https://baijiahao.baidu.com/s?id=1595349117525189591&wfr=spider&for=pc  

https://blog.csdn.net/drose29/article/details/92830231?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.add_param_isCf&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.add_param_isCf




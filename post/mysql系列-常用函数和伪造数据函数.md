---
title:       "常用方法和伪造数据函数"
subtitle:    ""
description: ""
date:        2019-08-29
author:      "麦子"
image:       "https://c.pxhere.com/images/e8/66/3d717bad8449de9495d9c9761d57-1423465.jpg!d"
tags:        ["mysql系列", "常用方法", "伪造数据函数"]
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


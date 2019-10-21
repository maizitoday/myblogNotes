---
title:       "mycat入门"
subtitle:    ""
description: ""
date:        2019-03-27
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-05-23-external_system_auth/background.jpg"
tags:        ["mycat","mycat","数据库","主从复制-mycat","读写分离"]
categories:  ["Tech" ]
---



[TOC]

 

### 数据切分

数据的切分（Sharding）根据其切分规则的类型，可以分为两种切分模式。一种是按照不同的表（或者 Schema）来切分到不同的数据库（主机）之上，这种切可以称之为数据的垂直（纵向）切分；另外一种则是根据 表中的数据的逻辑关系，将同一个表中的数据按照某种条件拆分到多台数据库（主机）上面，这种切分称之为数 据的水平（横向）切分。



### 逻辑表和物理表

逻辑表：逻辑表可以理解为数据库中的视图，是一张虚拟表。可以映射到一张物理表，也可以由多张物理表组成，这些物理表可以来自于不同的数据源。

物理表：物理表是具体某个数据源中的一张表。



### 为什么要用MyCat

转自： <https://blog.csdn.net/nxw_tsp/article/details/56277430>

***一、什么是MyCat： 
MyCat是一个开源的分布式数据库系统，是一个实现了MySQL协议的服务器，前端用户可以把它看作是一个数据库代理，用MySQL客户端工具和命令行访问，而其后端可以用MySQL原生协议与多个MySQL服务器通信，也可以用JDBC协议与大多数主流数据库服务器通信，其核心功能是分表分库，即将一个大表水平分割为N个小表，存储在后端MySQL服务器里或者其他数据库里。

MyCat发展到目前的版本，已经不是一个单纯的MySQL代理了，它的后端可以支持MySQL、SQL Server、Oracle、DB2、PostgreSQL等主流数据库，也支持MongoDB这种新型NoSQL方式的存储，未来还会支持更多类型的存储。而在最终用户看来，无论是那种存储方式，在MyCat里，都是一个传统的数据库表，支持标准的SQL语句进行数据的操作，这样一来，对前端业务系统来说，可以大幅降低开发难度，提升开发速度

所以可以这样理解：数据库是对底层存储文件的抽象，而Mycat是对数据库的抽象。他是所有背后数据库的一个管理者。 也是代码和数据库的一个接口。 



### mysql主从复制(docker)

1. mysql配置文件修改

```yaml

#拉取mysql镜像，进入容器，修改/etc/mysql/my.cnf

#在master中这个文件加入
[mysqld]
log-bin=mysql-bin #开启二进制日志
server-id=1 # 唯一服务器ID，非0整数，不能和其他服务器的server-id重复

#在slave中这个文件加入
[mysqld]
log-bin=mysql-bin
server-id=2

#这里面还有很多配置， 有需要在查询配置。

配置好后， 重新启动容器
```

2. 客户端配置主从关系

```yaml
#master
    #用数据库客户端工具连接master数据，执行一下数据，
    #主要作用： 创建同步账号，允许访问的IP地址为%，%表示通配符
    GRANT REPLICATION SLAVE ON *.* to 'root'@'%' identified by '123456';  
    
    #查看状态，记住File、Position的值，在Slaver中将用到
    show master status;
    
#slave    
   #用数据库客户端工具连接slave数据，执行一下数据，
   #主要作用： 设置主库链接
   #主机ip， 不能是127.0.0.1, 如果是的话slave_i_state会是null
   #MASTER_LOG_FILE 和 MASTER_LOG_POS 这个值，在master上面语句中有
   #port是容器端口
   CHANGE MASTER TO MASTER_HOST ='192.168.70.102', MASTER_USER ='root', MASTER_PASSWORD ='123456', MASTER_LOG_FILE ='mysql-bin.000001', MASTER_LOG_POS =429, MASTER_PORT =3306;
    
    #启动从库同步
    start slave;
    
    #查看状态
    show slave status;
    
 #成功的标志，
    根据上面的查看状态语句，出现了 
    slave_io_state:Waiting for master to send event
    slave_sql_running:Yes
    slave_io_running:Yes
    表示已经成功了，现在主从复制已经完成
```



### mycat实现读写分离

 主要是两个文件的修改， 一个是server.xml和schema.xml文件。 

#### schema.xml

```xml
<mycat:schema xmlns:mycat="http://io.mycat/">
   1. 这里定义了一个逻辑库，这个workdb是数据库集群对外的一个入口， 这个逻辑库对应的数据分片就是dn1
   <schema name="workdb" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1"></schema>
   2. 这个数据节点dn1,表示了一个数据库代理，同时他代表的真实的数据库名称是work，如果你在客户端创建新的schemar的时候，可以创建，但是这个schema不可以使用，这里只能对work进行处理。
  <dataNode name="dn1" dataHost="vm3306" database="work" ></dataNode>
        <dataHost name="vm3306" maxCon="1000" minCon="10" balance="3" writeType="0" dbType="mysql" dbDriver="native">
          3.这个是用于监控服务是正常的
			    <heartbeat>select user()</heartbeat>
			    <writeHost host="hostM1" url="192.168.70.102:3306" user="root" password="123456">
			        <readHost host="hostS1" url="192.168.70.102:3307" user="root" password="123456"/>
			    </writeHost>
      </dataHost>
</mycat:schema>

负载均衡类型：

（1）balance="0", 不开启读写分离机制，所有读操作都发送到当前可用的writeHost 上。

（2）balance="1"，全部的 readHost 与 stand by writeHost 参与 select 语句的负载均衡，简单的说，当双主双从模式(M1->S1，M2->S2，并且 M1 与 M2 互为主备)，正常情况下，M2,S1,S2 都参与 select 语句的负载均衡。

（3）balance="2"，所有读操作都随机的在 writeHost、 readhost 上分发。

（4）balance="3"，所有读请求随机的分发到 wiriterHost 对应的 readhost 执行，writerHost 不负担读压力，注意 balance=3 只在 1.4 及其以后版本有，1.3 没有。


writeType="0", 所有写操作发送到配置的第一个 writeHost，第一个挂了切到还生存的第二个writeHost，重新启动后已切换后的为准，切换记录在配置文件中:dnindex.properties

```



#### server.xml

```xml
配置可以处理的用户，同时这个schemas这个地方就是schema中的逻辑库

<user name="root">
		<property name="password">123456</property>
		<property name="schemas">workdb</property>
		
		<!-- 表级 DML 权限设置 -->
		<!-- 		
		<privileges check="false">
			<schema name="TESTDB" dml="0110" >
				<table name="tb01" dml="0000"></table>
				<table name="tb02" dml="1111"></table>
			</schema>
		</privileges>		
		 -->
	</user>

	<user name="root_work">
		<property name="password">123456</property>
		<property name="schemas">workdb</property>
		<property name="readOnly">true</property>
	</user>
```



如此就可以实现了mysql的读写分离，如图：

![](/img/mycat.png)



#### 测试是否读写分离

​        用客户端连接mycat服务器， 这个地方需要注意，我们连接的数据库的名称是work，同时，他的端口应该是8066/tcp这个端口， 我这边用的是docker映射出来的端口。 

1. 连接成功mycat的， 然后直接在里面创建表， 写入数据，如果看到master和slave都有数据，证明写是ok的。 

2. 然后在slave客户端手动添加一条数据， 然后在mycat的客户端直接查询这个表，查看出来的数据是否比master的数据要多一条，如果是多一条了，那么证明在查询的时候，是走的slave端。 

3. 需要主要，手动插入slave后，在添加数据，发现master和slave数据无法同步了， 这个时候，请查看在slave中执行 show slave status;  查看 

   slave_io_state:Waiting for master to send event
   slave_sql_running:Yes
   slave_io_running:Yes

   这些状态是否正常，如果不正常， 就百度处理，把他们修改正常。 因为slave是不可以写入数据的。 

### 自动切换主从

 当主机挂掉后，需要自动从机为主机，这个时候，我们只要让这两台机子互为主从，上面的操作在执行一次，这个时候，我们用mycat来配置，就可以实现当主机挂掉了，从机就是主机了。
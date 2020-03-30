---
title:       "Linux常用命令操作"
subtitle:    ""
description: ""
date:        2019-04-09
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-11-service-mesh-vs-api-gateway/background.jpg"
tags:        ["shell", "linux常用命令"]
categories:  ["Tech" ]
---

[TOC]

# 目录介绍

```shell
/home：
用户的主目录，在Linux中，每个用户都有一个自己的目录，一般该目录名是以用户的账号命名的。  

/opt：
 这是给主机额外安装软件所摆放的目录。比如你安装一个ORACLE数据库则就可以放到这个目录下。默认是空的。

/root：
该目录为系统管理员，也称作超级权限者的用户主目录。

/tmp：
这个目录是用来存放一些临时文件的。

/usr：
 这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似于windows下的program files目录。

/var：
这个目录中存放着在不断扩充着的东西，我们习惯将那些经常被修改的目录放在这个目录下。包括各种日志文件。
```

![4-9-1](/img/4-9-1.png)



![4-9-2](/img/4-9-2.png)

![4-9-3](/img/4-9-3.png)

# 常用的Linux命令

```shell
     ll:    -t   排序显示    -r倒序显示   -h 显示大小  
    cat:    输出文件中的所有内容，也可以输出多个文件。 -b 显示出非空的行号。 -n 显示出所有的行号
    tac:    倒序打印出文件
    rev:    把所有的内容都反过来显示。 如：maizi变成iziam
     wc:    -l 显示这个文件一共多少行， -m显示这个文件一共多少个字节。 
     cp:    拷贝， -r的时候拷贝一个目录，递归处理。 -v 就是显示出你拷贝的过程。
  mkdir:    创建目录， -p 递归创建目录， -v 显示创建过程。
     mv:    重命名和移动文件， 注意，如果是移动到当前目录，需要./这样， 默认是面对的是绝对目录
     du:    查看文件的大小  -h 显示里面每一个文件大小   -sh 显示文件的总大小
dirname:    获取当前文件的前面的目录。 
            如：dirname /home/maizi/maizi_today  得到的结果就是dirname/home/maizi
basename:   获取就是当前目录的最后一个目录： 
            如：  dirname /home/maizi/maizi_today   得到的结果就是： maizi_today
   echo:    重定向,可以把一些文本输出到一个文件中  
            如：echo -e "maizi_today" > b.txt，
            (使用>可以将本来出现在屏幕的标准输出信息重定向到一个文件中。
            用>>可以在实现重定向时不覆盖原有内容，而是在文件末尾追加内容)
```

# 字符操作

```shell
   tr:   替换或删除字符
  seq:   序列，  -s 指定换行符  -w 等宽，用0来填充   还可以指定步长   swq   1 2  10 
 shuf:   生成随机序列   shuf -i 5-10    shuf -i  5-20 | head -1 获取其中一个
 uniq:   去掉相邻的重复的， 可以先排序然后去重。
 join:   用于将两个文件中，指定栏位内容相同的行连接起来。类似于表的数据相互连接。
  cut:   选取文件中的每一行数据    
         -b  选中第几个字符  -c:选中多少个字符 -d：分隔符  -f：选中字段   
         下标从1开始(备注： 这个命令主要是针对截取字符串的时候使用)
 sort:   排序,-g  : 按常规数值排序   -o :指定输入位置  -u: 去重复行   -r  倒序排序    
          -n 根据字符串值比较   -h   根据大小排序  
         如： sort -h 1.txt  2.txt   如： du -h |   sort -hr
  tee:   指令会从标准输入设备读取数据，将其内容输出到标准输出设备，同时保存成文件。  
         -a或--append 附加到既有文件的后面，而非覆盖它．（备注：字符串处理）
```

# 查看

```shell
stat:   显示文件或者文件的系统状态 ， 修改时间， 创建时间， 权限。
head:   显示文章的前面几行，
        -n  显示文章的前面几十行        head -20 krb5.conf | cat -n   这样可以看到行号
tail:   查看文章末尾的数据，  +n 显示文本的最后多少行   -f 只要文本在更新，就一直显示后面的数据。 
  nl:   显示行号，和 cat -n 一样的效果
date:   这个时间是UTC时间， 比我们本机时间要快8个小时。 可以进行设置为本地时间。 
find：  搜索文件目录和结构 ，适合通配符。 
        -name ： 文件名，文件夹一样也可以
         -type： 文件类型， d目录，f常规文件 
         路径               文件名             动作 
         find /home/maizi  -name  "*.tar*"           
         find /home/maizi  -name  "*.tar*"   -ls       
```

# 下载

```shell
 wget:   直接下载 
 curl:    -o  index.html http://wwww.baidu.com  直接保存这个网页到index.html中。
        （备注：curl是一个利用URL规则在命令行下工作的文件传输工具，
               可以说是一款很强大的http命令行工具。它支持文件的上传和下载，
               是综合传输工具，但按传统，习惯称url为下载工具） 
```

# 系统相关硬件设置数据 

```shell
  free:   查看内存使用率   -m M显示   -g  G显示  -h 易读单位显示
    df:   查看系统磁盘空间使用情况   -h 易读单位显示， -t 只显示指定的文件类型
iostat:   CPU利用率和I/O   -c 显示CPU使用率 
   sar:   查看系统资源中和利用率   
          sar -u 2 3   每两秒一共监控3次 ，  -u CPU  -r 内存   -o 输入到文件中，用于分析
```

# 网络 ， 查看进程 ，端口

```shell
netstat:   命令用于显示网络状态。  -l  显示所有监听   -t 显示TCP    -u 显示udp信息   
           如果启动很多服务就可以查看了。  
           netstat -ap | grep ssh     找出程序运行的端口  
           netstat -anpt | grep ':8080'    找出运行在指定端口的进程
           -n 拒绝显示别名，能显示数字的全部转化成数字。
     ss:   查看socket工具。    ss -anpt | grep ssh     grep  -i  忽略大小写  
   lsof:   列出打开的文件。
           lsof -i tcp   监听网络地址     
           lost -i  :mysql   lost -i :22  lost -i :80 （查看这个端口）
     ps:   查看当前的进程快照   
           -a: 显示所有进程   -u  选择有效的用户id或者明灿   -e 显示所有经常  -x 显示无终端的进程  
            -p 显示指定进程   -f ：做一个更为完整的输出  ps -ef 是用标准的格式显示进程的、
            其格式如下， 常用都是 ps -elf 
    top:    动态显示活动的进程和系统资源利用率。 -b  将输出编排成易处理的格式   -H 显示线程   
     nc:    TCP 和 UDP的连接和监听。nc -v -z -w2 192.168.0.3 1-10000   连接2秒，查看端口
```

# 推送和拉取

```shell
    scp:  基于ssh的安全远程服务器拷贝 
          sup -p 22 -r src_dir root@192.168.1.10:/dst_dir 
          本地文件拷贝到远程目录 （两个Linux数据拷贝） 推送
          scp -p 22 -r  root@192.168.1.10:dst_dir src_dir 
          将远程主机目录拉取到本地
   rsync: 远程或者本地文件同步工具。  linux 上安装并配置 rsync，然后还要启动。
          推送： rsync  -avz src root@192.168.1.10:src 
          拉取： rsync -avz root@192.168.1.10:src src  
          -a:    递归，保留权限，属组。 
          -v:    显示复制信息
          -z:    压缩传输数据 
          指定端口： 
   rsync -avz  /Users/maizi/Desktop/run.sh -e "ssh -p10001"  root@192.168.70.102:home/maizi
```

# 解压和压缩 

```shell
tar：解压和压缩 
     -c:     创建新归档
     -x:     提取归档所有文件
     -f：    使用归档文件或者设备归档
     -z:     gzip压缩
     -v      输出处理过程 
     -t      存档的内容列表 
     -C      指定目录 
     tar   -cf    b.tar b.txt  
     tar   -tvf  b.tar
     tar   -xf    b.tar    解压
     tar -cvzf b.tar.gz b.tar   压缩b.tar为gz文件 
     tar -zxvf b.tar.gz  解压这个gz文件
```

# 针对系统文件进行修改

```shell
source：    source ~/.bashrc   这个波浪号代表是什么             ~代表主目录
通常用法：source filepath 或 . filepath
功能：使当前shell读入路径为filepath的shell文件并依次执行文件中的所有语句，通常用于重新执行刚修改的初始化文件，使之立即生效，而不必注销并重新登录。
例如，当我们修改了/etc/    profile文件，并想让它立刻生效，而不用重新登录，就可以使用source命令，如source /etc/profile。

   sudo:     Linux sudo命令以系统管理者的身份执行指令，也就是说，
             经由 sudo 所执行的指令就好像是 root 亲自执行。
             
history:     查看所有的命令历史记录  
```

# 权限 

```shell
id  root      查看一个用户  
useradd  maizi      用户添加 
userdel  -r maizi   用户删除 

#把一个用户加入到另外一个组 
gpasswd -a  maizi   root 
#把组中的用户删除 
gpasswd -a  maizi   root

chomd:
      chomd    o+w         o : other  g:group     
      chomd    g+w    other 其他人有writer的权限
      a+x 是给所有人加上可执行权限，包括所有者，所属组，和其他人
      o+x 只是给其他人加上可执行权限
      -R, --recursive：对目录以及目录下的文件递归执行更改权限操作。
      
Linux用户
    1、所有者（u）
    2、所属组（g） （所有者及所有者所在组的全部用户）
    3、其他用户（o）（其他组的所有用户（包括文件所有者））
    4、所有用户（a）
        r       4
        w       2
        x       1
```

# acl

```shell
setfacl:  设置文件权限——————————这个针对的是文件的内容的权限操作。
          setfacl -m   u:user1:rw  root.txt 
          设置user1这个用户有rw权限，对应的事root.txt这个文件
getfacl:  查看权限     
setback:  删除权限   setback -x   u:user1  root.txt    
          清空文件权限   setback -b    root.txt
          如果需要是对应的文件夹里面的文件的创建删除的时候， 我们需要的就是对上面一层目录处理。
          setfacl -m u:user1:rwx /mnt 
          
设置目录以及子目录和文件设置权限
setfacl -m u:user1:rwx  -R /mnt/   递归处理

 如果后面再创建文件和目录， 这个时候又是没有权限的， 这个时候需要如何处理
 setfacl -m d:u:user1:rwx  -R /mnt/    default  默认 ， 就是继承设置好的文件权限。
```

# 查找正在运行的进程

```shell
ss -nlpt   查看现在正在使用的TCP应用应用。 
netstat -ntlp  也是一样
ps -elf | grep ssh  查看正在运行的进程，这样可以找到这个进程的id，然后kill  pid，然后在重启服务。
```

# 命令登录ssh

```shell
终端利用ssh登录远程服务器安装ssh：
 启动ssh：
 service sshd start
 登录远程服务器：
 ssh -p 50022 my@127.0.0.1
 输入密码：
 #说明
 my@127.0.0.1:
 -p 后面是端口
 my 是服务器用户名
 127.0.0.1 是服务器 ip
 回车输入密码即可登录
```

# 解决中远程中文乱码问题

```shell
vi   ~/.bashrc  
export LANG='UTC-8' 
export LC_ALL='en_US.UTF-8'
#或者
export  LANG="zh_CN.UTF-8"
export  LC_ALL="zh_CN.UTF-8"
source ~/.bashrc  
#或者
sudo vim /etc/locale.conf
LC_ALL="en_US.utf8"
LC_CTYPE="en_US.utf8"
LANG="en_US.utf8"
```

# 防火墙和开机启动

```shell
#centos7版本
firewalld的基本使用
     启动:    systemctl start firewalld
     关闭:    systemctl stop firewalld
  查看状态:    systemctl status firewalld 
  开机禁用:    systemctl disable firewalld
  开机启用:    systemctl enable firewalld   
  

chkconfig --list #列出所有的系统服务
#启动nginx服务
systemctl start nginx.service
#设置开机自启动
systemctl enable nginx.service
#停止开机自启动
systemctl disable nginx.service
#查看服务当前状态
systemctl status nginx.service
#重新启动服务
systemctl restart nginx.service
#查看所有已启动的服务
systemctl list-units --type=service  
```

# 服务启动centos6版本

```shell
您的操作系统不使用systemd，systemctl但仍使用init.d或service命令：
例如：
sudo service {servicename} {stop|start|restart}
要么
/etc/init.d/{service} {stop|start|restart}
```

# rpm和yum的区别

```shell
RPM管理支持事务机制。增强了程序安装卸载的管理。

RPM的功能：打包、安装、查询、升级、卸载、校验、数据库管理。

由于Linux中的程序大多是小程序。程序与程序之间存在非常复杂的依赖关系。RPM无法解决软件包的依赖关系，但是yum下载的时候，可以把关联的软件包都可以下载了。
```

# chkconfig系统服务等级

```shell
chkconfig命令主要用来更新（启动或停止）和查询系统服务的运行级信息， 查看系统中的服务情况。

使用范例：
chkconfig --list #列出所有的系统服务
chkconfig --add httpd #增加httpd服务
chkconfig --del httpd #删除httpd服务
chkconfig --level httpd 2345 on #设置httpd在运行级别为2、3、4、5的情况下都是on（开启）的状态
chkconfig --list #列出系统所有的服务启动情况
chkconfig --list mysqld #列出mysqld服务设置情况
chkconfig --level 35 mysqld on #设定mysqld在等级3和5为开机运行服务，--level 35表示操作只在等级3和5执行，on表示启动，off表示关闭
chkconfig mysqld on #设定mysqld在各等级为on，“各等级”包括2、3、4、5等级

如何增加一个服务：
1.服务脚本必须存放在/etc/ini.d/目录下；
2.chkconfig --add servicename
在chkconfig工具服务列表中增加此服务，此时服务会被在/etc/rc.d/rcN.d中赋予K/S入口了；
3.chkconfig --level 35 mysqld on
修改服务的默认启动等级。  
```

# 重定向和管道

注明： 转载地址不记得了.....

重定向: 能够实现Linux命令的输入输出与文件之间重定向，以及实现将多个命令组合起来实现更加强大的命令

# 重定向标准输出

使用>可以将本来出现在屏幕的标准输出信息重定向到一个文件中

用>>可以在实现重定向时不覆盖原有内容，而是在文件末尾追加内容.

# 重定向标准错误信息到文件

没有单纯的一个操作符可以将标准错误信息重定向到文件中。要实现这一点有两种办法

 第一种使用文件描述符，在shell中，默认用数字0，1，2分别代表标准输入、标准输出、标准错误

# 屏蔽不想看到的信息

 Linux中有一个特殊的文件/dev/null，这个文件叫做bit bucket，

 可以接受输入信息但是什么都不做。因此要抑制命令的输出信息，只需要如下操作：ls /usr/ > /dev/null 

# 管道

将一个程序的标准输出写到一个文件中去，再将这个文件作为另一个程序的输入，管道要解决的就是不需要临时文件就   能将两条命令结合在一起。

- cat：   连接文件
- sort：  排序文本行
- uniq： 忽略或者报告重复行
- wc：    统计文件的行数、词数、字节数
- grep： 打印匹配制定模式的行
- head： 输出文件的头部
- tail：    输出文件的尾部
- tee：   从标准输入读，并往标准输出或者文件写

#  kill 

```shell
# 下面是常用的信号。
# 只有第9种信号(SIGKILL)才可以无条件终止进程，其他信号进程都有权利忽略。

HUP     1    终端挂断
INT     2    中断（同 Ctrl + C）
QUIT    3    退出（同 Ctrl + \）
KILL    9    强制终止
TERM   15    终止
CONT   18    继续（与STOP相反，fg/bg命令）
STOP   19    暂停（同 Ctrl + Z）
```

##  kill -9  和 kill的区别

1. kill -9 id：一般不加参数kill是使用15来杀，这相当于正常停止进程，停止进程的时候会释放进程所占用的资源；他们的区别就好比电脑关机中的软关机（通过“开始”菜单选择“关机”）与硬关机（直接切断电源），虽然都能关机，但是程序所作的处理是不一样的。

2. kill - 9 表示强制杀死该进程；而 kill 则有局限性，例如后台进程，守护进程等；

3. 执行kill命令，系统会发送一个SIGTERM信号给对应的程序。SIGTERM多半是会被阻塞的。kill -9命令，系统给对应程序发送的信号是SIGKILL，即exit。exit信号不会被系统阻塞，所以kill -9能顺利杀掉进程。

# telnet测试端口连通性

```shell
Telnet  ip  port   
```

```shell
$ telnet 101.199.97.65 62715
Trying 101.199.97.65...
Connected to 101.199.97.65.
Escape character is '^]'.
```

此时命令未退出。
根据提示Escape character is '^]'.可知退出字符为'^]'（CTRL+]）。此时输入其它字符不能使其退出，CTRL+C都不行。输入CTRL+]后会自动执行，进入命令模式：

```shell
^]
telnet>
```

此时再运行quit才会真正退出。

```shell
telnet> quit
Connection closed.
```
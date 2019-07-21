---
title:       "shell"
subtitle:    ""
description: ""
date:        2019-04-09
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/docker.jpg"
tags:        ["shell语法", "服务脚本"]
categories:  ["Tech" ]
---

[TOC]

/bin/bash    用这种来进行编译处理

### 变量

```shell
#变量
my_name="maizi_today"
echo $my_name

#只读变量 
myUrl="wwww.baidu.com"
readonly myUrl
echo ${myUrl}

#删除变量 
unset myUrl
echo "删除变量"
echo $myUrl
echo "删除变量成功"

#变量赋值需要带入$符号，不然就相当于字符串
temp_name=$my_name
echo $temp_name
```

 

### 加减乘除运算

```shell
#下面的不是单引号  加减乘除   进行四则运算的时候运算符号前后一定要有空格，乘法的时候需要进行转义
#如果是字符串形式的数据，会自己转为数字进行运算， 但是依然可以用 != 来比较字符串和数字类型。

a="1"
b=10

val=`expr $a + $b`
echo $val

val=`expr $b - $a`
echo $val

val=`expr $a \* $b`
echo $val


val=`expr $a / $b`
echo $val
```



### 判断语句

```shell
#  if
#  if-else
#  if-elseif
#  case

if [ $a == $b ]
then
   echo "a 和 b 相等 "
fi


if [[ $a != $b ]]; then
     echo "a 和 b 不相等"
fi


x=10
y=20
if [[ $x != $y ]]; then
    echo "x 和 y 不相等 "
fi


#注意这个then只有在if 和 elif才有， else是没有的
if [ $x == $y ]
then
    echo "x 和 y 相等"
elif [ $x != $y ]
then
    echo "x 和 y 不相等"         
else 
    echo "x 和 y 未知"
fi



# case 进行分支判断
case $1 in
    start)
        echo "程序启动成功"
        ;;
    stop)    
        echo "程序停止"
        ;;
       *)
      echo "程序正在部署中"
        ;;    
esac
```



### 关系运算符

```shell
#这个只能用来比较数字类型或者数字类型的字符串，如果是字符串无法进行比较

#   -eq 两个数相等返回true
#   -ne 两个数不相等返回true
#   -gt 左侧数大于右侧数返回true    
#   -lt 左侧数小于右侧数返回true
#   -ge 左侧数大于等于右侧数返回true
#   -le 左侧数小于等于右侧数返回true

aa="10"
bb="100"

if [[ $aa -eq $bb ]]; then
     echo "关系运算符: aa 和 bb 相等"
fi


if [[ $aa -ne $bb ]]; then
     echo "关系运算符: aa 和 bb 不相等"
fi
```



### 字符串运算

```shell
#   =    两个字符串相等返回true
#  !=    两个字符串不相等返回true
#  -z    字符串长度为0返回true    
#  -n    字符串长度不为0返回true


#  -d file    检测文件是否是目录，如果是，则返回 true 
#  -r file    检测文件是否可读，如果是，则返回 true
#  -w file    检测文件是否可写，如果是，则返回 true
#  -x file    检测文件是否可执行，如果是，则返回 true
#  -s file    检测文件是否为空（文件大小是否大于0，不为空返回 true
#  -e file    检测文件（包括目录）是否存在，如果是，则返回 true
 
如： 

filename="/tmp/centosConfig/a.sh"
if [ -d "$filename" ]; then
  echo "$filename is a directory "
elif [ -f "$filename" ]; then
  echo "$filename is a file"
fi


#字符串方法
temp="maizi_today"
# 输出字符串长度
echo ${#temp}
# 截取字符串,下标从0开始
echo ${temp:0:5}
```



### 数组

```shell
array=(1 2 3 4 5 6) #定义数组 
array2=(aa bb cc dd ee) #定义数组 
#获取某一个值
echo ${array[0]}
val=${array[1]}
echo $val

#获取长度 
length=${#array[*]}
echo $length

count=${#array[@]}
echo $count
```



### 循环

```shell
#    支持一维数组 ，没有限制数组的大小
#    在 Shell 中，用括号来表示数组，数组元素用"空格"符号分割开。定义数组的一般形式为：

array_data=(1 "12345678" 3 "4")
echo ${array_data[1]}


for i in {1..5}; do
    echo $i
done


for langue in java android ios; do
     echo "${langue}"_name
done


for (( i = 0; i < 10; i++ )); do
    echo $i
done


for file in $(ls /etc); do
    echo  ${file}
done


for file in $HOME/.bash*; do
    echo $file
done


#获取长度
echo ${#array_data[@]}
echo ${#array_data[1]}
```



### 函数

```shell
sysout(){
    echo "我是函数"
}
sysout

#有返回值   $?接收上一程序的返回值状态
test(){
    return 5 
}
test
result=$?
echo $result

#定义一个需要参数的 
show(){
    echo $1
    echo $2
}
show "maizi_today"  "yub"
```



### 重定向 

```shell
# $echo result > file  #将结果写入文件，结果不会在控制台展示，而是在文件中，覆盖写
# $echo result >> file  #将结果写入文件，结果不会在控制台展示，而是在文件中，追加写
# echo input < file  #获取输入流

ls -l >yubo.txt    直接重定向 


让控制台输入
read  -p  “please input your name ”  name1 

```



### shell里面执行Linux命令

```shell
 ``  这个符号可以接收命令后执行的结果，   str =`date`， 单引号里面不要放入$符号
 同时也可以用 $(ls)  这样也可以在shell中执行Linux命令。
```



### 函数只返回数据，不会返回字符串

```shell
 Shell 函数返回值只能是整形数值，一般是用来表示函数执行成功与否的，0表示成功，其他值表示失败。
 因而用函数返回值来返, 回函数执行结果是不合适的。如果要硬生生地return某个计算结果，比如一个字符串，
 往往会得到错误  提示：“numeric argument required”。
```



### 传递参数和预定义变量

```shell
echo "Shell 传递参数实例！";
echo "第一个参数为：$1";
echo "参数个数为：$#";
echo "传递的参数作为一个字符串显示：$*";
echo "传递的参数作为多个字符串显示：$@"
echo "脚本运行的当前进程ID号"
```



![屏幕快照 2019-04-09 下午9.42.24](/img/4-9-5.png)



### Linux nohup、&、 2>&1是什么

**原文地址**：**https://blog.csdn.net/lovewebeye/article/details/82934049**

基本含义

- /dev/null 表示空设备文件
- 0 表示stdin标准输入
- 1 表示stdout标准输出
- 2 表示stderr标准错误



>file 表示将标准输出输出到file中，也就相当于 1>file

2> error 表示将错误输出到error文件中

2>&1 也就表示将错误重定向到标准输出上

2>&1 >file ：错误输出到终端，标准输出重定向到文件file，**等于 > file 2>&1(标准输出重定向到文件，错误重定向到标准输出)。**

& 放在命令到结尾，表示后台运行，防止终端一直被某个进程占用，这样终端可以执行别到任务，配合 >file 2>&1可以将log保存到某个文件中，但如果终端关闭，则进程也停止运行。如 command > file.log 2>&1 & 。

**nohup放在命令的开头，表示不挂起（no hang up），也即，关闭终端或者退出某个账号，进程也继续保持运行状态，一般配合&符号一起使用。如nohup command &。**

### &` 号

所以，可以在命令的末尾加上一个 `&` 号，将这个任务放到后台去执行：

### 常用脚本列子

```shell
#!/bin/sh

## java 此处是指定jdk启动
export JAVA_HOME=/opt/java/jdk1.8.0_201
export JRE_HOME=$JAVA_HOME/jre

##此处是打包的jar包名称，不带.jar后缀
API_NAME=springbootdemoweb-0.0.1-SNAPSHOT
JAR_NAME=$API_NAME\.jar

#PID  代表是PID文件
PID=$API_NAME\.pid


#使用说明，用来提示输入参数
usage() {
    echo "Usage: sh 执行脚本.sh [start|stop|restart|status]"
    exit 1
}


#检查程序是否在运行
is_exist(){
  pid=`ps -ef|grep $JAR_NAME|grep -v grep|awk '{print $2}' `
  #如果不存在返回1，存在返回0
  if [ -z "${pid}" ]; then
   return 1
  else
    return 0
  fi
}


#启动方法
start(){
  is_exist
  if [ $? -eq "0" ]; then
    echo ">>> ${JAR_NAME} is already running PID=${pid} <<<"
  else
    nohup $JRE_HOME/bin/java -Xms256m -Xmx512m -jar $JAR_NAME >springboot.log 2>&1 &
    echo $! > $PID
    echo ">>> start $JAR_NAME successed PID=$! <<<"
   fi
  }


#停止方法
stop(){
  #is_exist
  pidf=$(cat $PID)
  #echo "$pidf"
  echo ">>> api PID = $pidf begin kill $pidf <<<"
  kill $pidf
  rm -rf $PID
  sleep 2
  is_exist
  if [ $? -eq "0" ]; then
    echo ">>> api 2 PID = $pid begin kill -9 $pid  <<<"
    kill -9  $pid
    sleep 2
    echo ">>> $JAR_NAME process stopped <<<"
  else
    echo ">>> ${JAR_NAME} is not running <<<"
  fi
}

#输出运行状态
status(){
  is_exist
  if [ $? -eq "0" ]; then
    echo ">>> ${JAR_NAME} is running PID is ${pid} <<<"
  else
    echo ">>> ${JAR_NAME} is not running <<<"
  fi
}

#重启
restart(){
  stop
  start
}

#根据输入参数，选择执行对应方法，不输入则执行使用说明
case "$1" in
  "start")
    start
    ;;
  "stop")
    stop
    ;;
  "status")
    status
    ;;
  "restart")
    restart
    ;;
  *)
    usage
    ;;
esac

#rm -rf $JAR_NAME

exit 0
```


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
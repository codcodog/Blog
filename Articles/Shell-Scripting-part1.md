---
title: Shell Scripting part1
date: 2016-09-12 23:16:19
tags: 
- Shell
- Linux
categories: Linux
---

#### shell编程基础原理和必备知识点 ####
一个字节 = 8位（字节=8bit）

程序：  
编译语言(静态语言)，编译执行，C，C++  
变量类型为强类型。

脚本语言（动态语言）,依赖于解释器，解释执行  
变量类型为弱类型。(默认为字符型)  

JAVA,垃圾回收器,会自动回收内存,因此执行效率相对C来说不高效.而C,不会自动释放内存，需要手动释放，容易造成内存被占用。

对于静态语言：  
源程序-->编译-->链接-->执行  

对于脚本语言：  
源程序-->使用解释器解释执行  

bash(解释器), perl, python, php, ruby(日本人开发的)  
bash相当于一种脚本语言.  

#### shell编程基础环境变量和编程符号应用 ####
bash变量类别：  
本地变量  
> 作用范围仅限当前源程序文件  
> 仅对当前shell进程有效,对子shell无效  

环境变量  
特殊变量  
位置变量  

`echo -n` 连接两个字符.  
`echo -e` 让转译符生效.(\t,\n)  

bash中的引号：引用  
单引号: 强引用  
双引号: 弱引用(可以变量替换)  
反引号: 命令替换  

变量替换：`echo '$age'`  

> shell可以有子shell: 在一个进程中，又打开一个新的shell.而本地变量对子shell是无效的,只对当前shell有效.

变量的声明和赋值：  
声明：  
declare AGE:  
`-l`: 声明为整型  
`-a`: 声明为数组  
`-r`: 声明变量为只读  

> 当声明为字符类型时，如果字符中间有空格的话，需要使用双引号，如果中间没有空格的话，单，双引号都行.  

引用变量值：  
${VARNAME}

撤销变量：  
unset VARNAME(注意不能加$符)  

只读变量(无法撤销，无法修改值)：  
readonly VARNAME

#### shell入门中变量赋值深入讲解 ####
变量名：字母，数字和下划线_  
环境变量：系统的变量，一般大写字母，例如：$SHELL  
``
VAR1 = $(VAR2 - word)  
VAR2存在则用VAR2,否,用word.
``

环境变量：  
作用范围是但前shell及其子shell.  
`export VARNAME=value` 声明为环境变量  
`declare -x VARNAME=value`  

查看所有环境变量：  
`env`  
`printenv`  
`export`  

``
PS1=[\u@\h \W]\$  
``

特殊变量：  
`$?` 命令的状态返回值.(0:执行成功,1-255:失败.其中,1,2,127系统留用)  

#### 变量中替换和bash的运行方式 ####
变量替换：用变量的值替换变量名，`echo "$AGE"`.  
命令替换：用命令的执行结果替换命令.`$(COMMAND)`  
``
echo "Today is `date +%F`"
``

`seq 5`: 1,2,3,4,5  

bash:  
变量：配置文件  
全局：  
`/etc/profile`, `/etc/profile.d/*`, `/etc/bashrc`  
个人：  
`~/.bash_profile`, `~/.bashrc`  
bash的运行方式：  
交互式:`/etc/profile`-->`/etc/profile.d/*`-->`~/.bash_profile`-->`~/.bashrc`-->`/etc/bashrc`  
非交互: `~/.bashrc`-->`/etc/bashrc`-->`/etc/profile.d/*`  

#### Linux中环境变量设置和开机变量加载 ####
profile类:  
设定环境变量  
运行命令或脚本  

bashrc类：  
设定本地变量  
设定命令别名  

`source 文件` 重新加载文件.  
`. 文件` 重新加载文件.  

shell脚本：  
命令的堆砌.  

shebang: 魔数，magic number.  
`#!/bin/bash`  

#### Linux输入输出和管道，重定向 ####
程序的默认输入设备，叫标准输入，stdin，默认是键盘. 0  
程序的默认输出设备，叫标准输出，stdout，默认minitor. 1  
程序的默认错误信息输出设备，标准错误输出，stderr，minitor. 2  

输出重定向：  
`COMMAND > FILE` 覆盖输出  
`COMMAND >> FILE` 追加输出  

set -C|+C `>|` 覆盖输出  

`/dev/null` 数据黑洞.  
`$?` 命令执行成功与否: 0，成功，1-255，失败.  
`/dev/zero` 泡泡机.  
`/dev/random`,`/dev/urandom` 随机数生成器.  

输入重定向：  
`COMMAND < FILE`  
`COMMAND << "EOF"` `EOF`: heredocument.  

错误输出：  
`COMMAND 2> FILE`  
`COMMAND 2>> FILE`  

正确与错误都重定向：  
`COMMAND > FILE 2> FILE2`  
`COMMAND > FILE 2> &1`  
`COMMAND &> FILE`  

管道：只传正确的信息.  
cut,tr,sort,uniq,tee

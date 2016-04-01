---
layout: post
title: 「Shell脚本学习指南笔记」第06章-变量-判断-重复动作
filename: 2014-04-15-notes-Classic-Shell-Scripting-chapter-06.md
category: Unix
tags: [shell]
---

流程控制的功能造就了程序语言：如果只是命令语句，是不可能完成任何工作的。本章介绍了用来测试结果、根据结果作出判断、加入循环、函数的功能。

<!-- more -->

## 变量与算术

------

内嵌（inline）算术
 
### 变量赋值与环境

```bash
export                              修改或打印环境变量 P123
export PATH=$PATH:/usr/local/bin    添加新变量到环境。
export -p                           打印当前环境。
PATH=/usr/bin awk ...               **改变awk命令所继承的环境内的、shell的内置变量。** P156
readonly variable ...               设置环境变量只读。
evn -i PATH=$PATH awk ...           i（initializ）忽略继承的环境，仅使用命令行指定的变量。
unset [-v] variable ...             删除当前shell的变量
unset -f function ...               删除当前shell的函数
```
 
### 参数展开（parameter expansion）
 
```bash 
P127 表6-1：替换运算符
运算符              含义：如果varname存在且非NULL，则
${varname:-word}    返回其值；否则，返回word。用于未定义时返回预设值。
${varname:=word}    返回其值；否则，赋值为word。
${varname:?word}    返回其值；否则，显示 varname:message,并退出当前命令或脚本。用于捕捉错误。
${varname:+word}    返回word；否则，返回NULL。用于测试变量存在。
```
 
位置参数（positional parameter），又称命令行参数
 
```bash 
P132 表6-3：POSIX内置的shell变量
变量                含义
#                   当前进程的命令行参数个数。
@                   当前进程的所有命令行参数。加了双引号"$@", 可以引用每一个参数。
*                   当前进程的所有命令行参数。加了双引号"$*", 相当于字符串。
- (means hyphen)    在引用时给予shell的选项。
?                   前一命令的退出状态。
$                   shell进程的进程编号（process ID）。
0 (means zero)      shell程序的名称。
!                   最近一个后台命令的进程编号。wait命令可使用!存储的编号。
ENV                 $ENV可展开，结果为读取和启动时要执行的文件的完整路径名称。
HOME                根目录（登录目录）。
IFS                 内部的字段分割器。一般设置为空格、制表符、换行符。
LANG                当前locale的默认名称。
LC_ALL              当前locale的名称。
LC_COLLATE          用来排序字符的当前locale名称。
LC_CTYPE            在模式匹配期间，用来确定字符类别的当前local名称。
LC_MESSAGES         输出信息的当前语言名称。
LINENO              刚执行过的行，在脚本或函数内的行号。
NLSPATH             信息目录的位置。
PATH                环境变量（命令查找的路径）。
PPID                父进程的进程编号。
PS1                 主要的命令提示符，默认为"$"。
PS2                 行继续的提示符，默认为"> "。
PS4                 以 set -x 设置的执行跟踪的提示字符串，默认为"+ "。
PWD                 当前工作目录。
```
 
### 算术展开（arithmetic expansion）
shell算术运算符与优先级，与C类似。运算符置于$((...))语法中，无需反斜杠转义（除了内嵌双引号外）。
 

 
## 退出状态

------

以惯例来说，退出状态为0表示“成功”，其它任何值表示失败。P134
`$?`是前一命令的退出状态。
`exit [exit-value]` 从shell返回一个退出状态给脚本的调用者。
 
### if-elif-else-fi语句
不要尝试过度“简练”而使用`&&`和`||`取代if语句。我们不反对简短且简单的事情，比如：
 
```bash 
$ who | grep weiyi >/dev/null && echo weiyi is logged on.
weiyi is logged on.
```
    
实际是执行`who | grep`如果成功，就显示信息。而如下：
 
```bash 
some_command && {
    first_cmd
    second_cmd
    third_cmd
}
```

花括号将所有命令框在一起，只有在some_command成功时，才被执行。使用if更易读：
 
```bash 
if some_command
then
    first_cmd
    second_cmd
    third_cmd
fi
```

一个例子：

```bash 
#!/bin/sh

# get day(1~7) of week
day=$(date +%u)

# if day less than 6, then it's a workday
if [ $day -lt 6 ] ; then
    echo ++--Working Hard--++
else
    echo ++--Sleeping Hard--++
fi
```
 
### test命令
`test [ expression ]` 或者 `[ [expression] ]`
测试文件属性、比较字符串、比较数字。通过退出状态返回结果。**第二种形式，必须与括起来的expression以空白隔开。** P138

```bash
表6-6：test表达式
运算符      如果......则为真
string      string不是null
s1 = s2     字符串s1与s2相同，**注意不是==**
s1 != s2    字符串s1与s2不同
n1 -eq n2   n1 equal n2
n1 -nq n2   n1 not equal n2
n1 -lt n2   n1 <  n2                less than
n1 -gt n2   n1 >  n2                greater than
n1 -le n2   n1 <= n2                less    & equal
n1 -ge n2   n1 >= n2                greater & equal
```
 
test使用注意事项
 
- 须有参数：
  test中的shell变量应该以引号括起来，这样test才能接收一个参数，即使是null字符串。
  `if [ -f "$file" ]`正确， `if [ -f $file ]`错误。
- 字符串比较的经验做法（compare string in unix）：
  当字符串为空，或开头带一个减号的时候，test命令会被混淆。所以，
  `if [ "X$answer" = "Xyes" ]`
- 只能作整数数字测试。
 
 
## case语句

------

根据模式列表依次匹配，匹配时执行相应的代码，直至`;;`为止。
模式里可包含任何的shell通配字符，且变量、命令、算术替换会在模式匹配之前被执行。P143
 
```bash
case $1 in                          # case in之间的shell变量，不必双引号
-f)
    ...                             # 针对选项-f的操作
    ;;
-d | --directory)                   # 允许长选项；允许“或 |”多个模式
    ...
    ;;                              # 针对选项-d的操作
*)                                  # default
    echo $1: unknown option >&2     # `>&2` 重定向到标准错误
    exit 1
    ;;
esac
```
 
## 循环
 
------
 
**for循环**
for用于循环对象列表（命令行参数、文件名、任何可以以列表形式建立的东西）。P144
列表缺省时，代表`for i in "$@"`.
 
```bash
for i in 对象列表
do
    ...
done
```
 
**while与until循环**
等待某个事件发生时，until循环很有用。
可以将管道放入while循环中，用来处理每一行的输入。也可以使用管道将循环的输出传递给另一个命令。P146
 
**break与continue**
 
**shift与选项的处理**
getopts TODO
 
 
## 函数
 
------

`return [eixt-value]`
如果函数要返回表示失败的非零值，应该设置一个全局shell变量，或利用父脚本捕捉它（使用命令替换7.6 TODO）。
 
所有的函数都与父脚本共享变量，所以，要留意误修改父脚本的内容。P152



## 版本
li2 于上海闸北 
2014-04-15 ~ 2014-05-15, v1

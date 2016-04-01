---
layout: post
title: 「Shell脚本学习指南笔记」第09章-awk的惊人表现
filename: 2014-04-15-notes-Classic-Shell-Scripting-chapter-09.md
category: Unix
tags: [shell]
---

awk可以胜任几乎所有的文本处理工作。
 
## awk命令行

------
 
 
## awk程序模型

------
 
```bash
pattern { action }  如果模式匹配，则执行操作
pattern             如果模式匹配，则打印记录
        { action }  针对所有记录，执行操作
```
 
awk以保留字BEGIN和END提供两种特殊模式。
 
- BEGIN { 在任何命令行文件或一般命令赋值被处理之前，在任何开头的-v选项指定完成之后。用来初始化 }
- END { 关联的操作也只执行一次，在所有输入数据已被处理完之后。用来产生摘要报告或者执行清除操作 }
   
<!-- more -->

## 程序元素

------
 
**字符串转数字**
 
**标量变量**
保存单一值的变量叫做标量变量。变量无需事先声明。在第一次被使用时被自动建立。使用变量时，内容中期待的是数字还是字符串就很清楚了，而且类型也可以在需要时转换。
新建的变量被初始化为一个空字符串，但需要数值时，它被视为0.
 
习惯上，局部变量全小写，全局变量首字母大写，内建变量全大写。
 
**内建标量变量**
 
| 变量      | 说明                              |
|-----------|-----------------------------------|
| FILENAME  | 当前输入文件的名称                |
| FNR       | 当前输入文件的记录数              |
| FS        | 字段分隔符，默认为空白符          |
| NF        | 当前记录的字段数                  |
| NR        | 在工作（job）中的记录数           |
| OFS       | 输出字段分隔符，默认为空白符      |
| ORS       | 输出记录分隔符，默认为换行符      |
 
 
**数组变量**
 
- 任意值索引，允许查找（find）、插入（insert）、删除（delete）操作，比如telephone["weiyi"]="4735"
- 数组无需声明，存储空间在引用新元素时自动增长，且是稀疏的（sparse），比如x[1]=3.14后接x[10]="ten"，而不必填满2~9;
- 数组元素类型可以不同；
- 元素不需要时，其存储空间可以回收再用，使用delete array[index]从数组中删除该元素。
 
**以逗号分隔的索引列表**
 
**命令行参数**
可对命令行自动化处理，与C、C++、Java、Shell不同。
内建变量ARGC、ARGV。
选项`--`停止参数解释。
 
**环境变量**
内建数组ENVIRON

```bash
$ awk 'BEGIN { print ENVIRON["HOME"]; print ENVIRON["USER"] }'
/home/weiyi
weiyi
``` 
 
## 记录与字段

------

**记录分隔字符**
记录是由一个或多个空行分隔的段落，位于文件起始或结尾处的空行会被忽略。或由内建变量RS指定的字符分隔。
 
**字段分隔字符**
字段由换行符，或内建变量FS指定的字符分隔。
 
 
 
## 模式与操作

------

**模式**

```bash 
NF==0                       选定空记录
NF>3                        选定拥有3个字段以上的记录
NR<5                        选定第1到4条记录
$1 ~ /jones/                选定字段1里有jones的记录
$0 ~ /[Xx][Mm][Ll]/         选定有xml的记录，忽略大小写
/[Xx][Mm][Ll]/              同上
```
 
使用范围表达式(range expression)
以逗点隔开两个表达式
`/[aeiouy]/ , /[^aeiouy]/` 选定起始于1个元音、结尾于1个辅音的记录
 
 
 
## 在awk里的单行程序

------
 
单词计数程序
 
```bash
$ echo "Hello World, I'm li2." | awk '{ C += length($0) + 1; W += NF } END { print NR, W, C }'
1 4 22
```
 
撇开NUL字符，awk可以代替cat
 
```bash
cat *.c
awk 1 *.c 产生相同的输出
```
 
从文本文件里，打印5%行左右的随机样本
 
```bash
# 使用虚拟随机产生函数，生成分布于0与1之间的值
awk 'rand() < 0.05' file
```
 
在以空白符为字段分隔符的表格里，统计第n个字段（即第n列）的和、平均值
 
```bash
$ cat sum.txt
1 2 3 4 5 6 7 8
2 3 4 5 6 7 8 9
$ awk '{sum += $1 } END { print sum }' sum.txt
3
$ awk '{sum += $2 } END { print sum }' sum.txt
5
$ awk '{sum += $2 } END { print sum/NR }' sum.txt
2.5
```
 
三种查找文件内文本的方法
 
```bash
egrep 'pattern1|pattern2' files
awk 'pattern1|pattern2' files
awk 'pattern1|pattern2 { print FILENAME ":" FNR ":" $0 }' files
```
 
 
仅查找2到10行
 
```bash
# 使用sed会漏掉位置信息，且需要选项-s为每个文件重新开始编号
$ sed -n -e 2,10p -s files | egrep 'pattern'
# 或者使用awk
$ awk '(FNR>=2) && (FNR<=10) && /pattern/ { print FILENAME ":" FNR ":" $0 }' files
```
 
 
调换第2列与第3列
 
```bash
# 假设以制表符分隔
awk -F'\t' -v OFS='\t' '{ print $1, $3, $2, $4 }' old > new
awk 'BEGIN { FS="\t" ; OFS="\t" } { print $1, $3, $2, $4 }' old > new
awk -F'\t' '{ print $1 "\t" $3 "\t" $2 "\t" $4 }' old > new
``` 
 
 
## 语句

------

程序语言必须支持连续性的、条件式的、重复的执行。
 
```bash
awk 'NR!=1 {
    if ($6 ~ /TIME|ESTABLISHED/)
        print > "coolshell-1.txt"
    else if ($6 ~ /LISTEN/)
        print > "coolshell-2.txt"
    else
        print > "coolshell-3.txt"
}' coolshell-netstat.txt
```
 
**数组成员测试**
成员测试`key in array`是一个表达式：如果key为array的一个值的索引，则表达式为真（1）。
否定测试`!(key in array)`，圆括号是必须的。
 
对于多下标（subscript）数组，测试时，使用圆括号，以逗点分隔下标：`(i, j, ..., n) in array`.
 
测试不会建立数组成员，但引用时，如果该成员不存在，便会建立它。
 
```bash
if ("weiyi" in telephone)        正确的测试方法
    print "weiyi is in the directory"

if (telephone["weiyi"] != "")    当weiyi不存在时，会将其加入数组索引中，并初始化为一个空串。
    print "weiyi is in the
```
 
**用户控制的输入**
 
**输出重定向**
 
 
## 用户自定义函数

------
 
## 字符串函数

------

**子串提取（字符串提取）**
`substr("abcde", 2, 3)` 返回"bcd"，**从1开始编号**。
 
**字母大小写转换**
`tolower("aBcDeF123")` 返回"abcdef123",
`toupper("aBcDeF123")` 返回"ABCDEF123".
 
**字符串查找**
`index("abcdef", "de")` 返回4.
`index(tolower(string), tolower(find))` 可以在查找时忽略大小写。
gawk提供了内建变量IGNORECASE，非零时忽略在字符串匹配、查找、比较时字母的大小写。
 
**字符串匹配**
 
**字符串替换**
 
**字符串分割**
 
**字符串重建**
 
**字符串格式化**
 
 
 
## 数值函数

------


## 版本
li2 于上海闸北 
2014-04-15 ~ 2014-05-15, v1

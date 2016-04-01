---
layout: post
title: 「Shell脚本学习指南笔记」第10章-文件处理
filename: 2014-04-15-notes-Classic-Shell-Scripting-chapter-10.md
category: Unix
tags: [shell]
---

本章讨论处理文件时常见的一些命令，如，列出文件、修改时间戳、建立临时文件、在目录层级中寻找文件、将命令应用到文件列表、计算文件系统空间的使用量、比较文件。

 
## 列出文件

------
 
```bash
ls -liRs *          显示文件（冗长的、inode编号、递归的、以块为单位列出文件的大小）
``` 
 
<!-- more -->

## 寻找文件

------

**快速寻找文件**
locate将系统里的所有文件名压缩成数据库，以迅速找到匹配shell通配字符模式的文件名，不必实际查找整个庞大的目录结构。这个数据库通常在半夜通过cron，在具有权限的工作中执行updatedb建立。locate可以回答用户：系统管理者将gcc包放在何处？P292
 
**寻找命令存储位置**
 
```bash
# 当你调用一个没有路径的命令时，可以通过Bourne-Shell家族里的type命令查找它在文件系统里的位置。
# type是内建shell命令
$ type awk
awk is hashed (/usr/bin/awk)
```

### find命令

`find [ files-or-directories ] [ options ]` 寻找与指定名称模式匹配的，或具有给定属性的文件。
**find与其它UNIX命令最大的不同之处在于：要查找的文件或目录放在参数列表第一位，选项或操作在命令行最后。**

```bash 
find | LC_ALL=C sort        以传统顺序，排序find的输出结果。
find -ls                    寻找文件，并用 ls 风格（各种文件信息的列表）的输出结果
find -ls | sort -k11        寻找文件，并以文件名排序
find $HOME/. ! -user $USER  在登录目录下，列出所有不属于我的文件。!表示非，选项-user指定某用户的文件。
find $HOME/. -size +1024k   在登录目录下，寻找大于1024k的文件。+表示大于，-表示小于，无符号表示等于。
find . -size 0              在当前目录下，寻找空文件。无单位是默认情况，表示512Bytes。
find . -type d              在当前目录下，寻找目录。选项-type表示文件类型，d目录，f文件，l符号连接。
find . -mtime -7            在当前目录下，寻找7天内修改过的文件。-atime访问时间，-ctime是inode变更时间。符号表示在7天内，+号表示超过7天，无符号表示整好7天。
find . -mtime +7 -o -size 0 支持布尔运算，选项-o（是字母o）表示“或OR”，选项-a表示“与AND”。
find path -name 'pattern'   选定与文件名与pattern（shell通配字符模式）匹配的文件。
```
 
 
## 执行命令：xargs

------

把find产生的文件列表提供给另一个命令，有时很有用。
通常由shell的命令替换功能`$()`完成，下例是在系统的头文件中查找POSIX_OPEN_MAX符号。
当使用命令处理find产生的对象列表时，如果列表为空，也应该确保命令的行为是适当的。因为未给定grep文件参数时，会等待标准输入。所以提供/dev/null以确保find未产生输出时，grep不会卡在等待输入。P307

```bash 
$ grep POSIX_OPEN_MAX /dev/null $(find /usr/include -type f | sort)
/usr/include/i386-linux-gnu/bits/posix1_lim.h:# define _POSIX_OPEN_MAX  20
```
 
来自命令替换产生的输出有时很长，如果超出环境变量规定的最大值`getconf ARG_MAX`，会提示"grep: Argument list too long."
xargs可以解决这个问题：它在标准输入上取得参数列表、一行一个，将它们以适当大小组起来传给另一个命令。
 
```bash
$ find /usr/include -type f | sort | xargs grep POSIX_OPEN_MAX /dev/null
``` 
 
 
## 文件系统的空间信息

------

`find -ls`可以报告文件大小，加上awk程序，可以得到文件占据的字节数：

```bash 
$ find -ls | awk '{ sum+=$7 } END { printf("Total: %.0f bytes\n", sum) }'
Total: 12291 bytes
```
 
然而文件以固定大小的块（block）配置，所以上述低估了空间的使用。它也不能提供整个文件系统已用空间、未用空间。df和du可以解决这些问题。
 
**df命令**

```bash 
# disk free 磁盘可用空间，提供单行摘要，每行显示一个加载的文件系统磁盘使用信息。
$ df
Filesystem     1K-blocks      Used Available Use% Mounted on
/dev/sda6      884533604 445397336 394204480  54% /home

# 选项-h表示human-readable
# 可以提供一个或多个文件系统名称或者加载点，来限制输出项目。
$ df -h /home
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda6       844G  425G  376G  54% /home
```
 
**du命令**

```bash 
# du命令可以提供指定目录树的空间使用情况，disk usage
# 选项-h表示human-readable，-s表示摘要summary
$ du -hs
12K     .
``` 
 
## 比较文件

------
 
- 检查两个文件是否不同，若不同，找出不同之处：cmp和diff
- 应用两个文件的不同之处，基于原始文件，为另一个文件打补丁：patch
- 使用校验和（checksum）找出相同一致的文件：消息摘要工具md5、md5sum，安全散列算法工具sha、sha1sum等
- 使用数字签名以验证文件。



## 版本
li2 于上海闸北 
2014-04-15 ~ 2014-05-15, v1

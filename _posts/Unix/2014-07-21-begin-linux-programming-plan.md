---
layout: post
title: 「Linux程序设计」学习计划跟踪
filename: 2014-07-21-Begin-Linux-Programming-Plan.md
category: Unix
tags: []
---

跟踪制定的学习计划。

## task4 done 2014-08-21

完成8.4节[使用mysql数据库的CD唱片应用程序](https://github.com/li2/beginning-linux-programming-exercise/tree/master/cd_mysql)的练习。


## task3 done 2014-07-29

完成7.4节[使用dbm数据库的CD唱片应用程序](https://github.com/li2/beginning-linux-programming-exercise/tree/master/cd_dbm)的练习。

- 在文件中以逗号或空格分隔变量来存储信息，在shell中容易实现。但许多CD标题和曲目有包含逗号。而使用数据库的索引机制可以放弃这种分隔方法：存储长度可变的数据记录；更有效地存储和检索数据。dbm是轻量级软件，可以被编译进二进制文件，无需独立的数据库服务器。
- 把数据访问接口和用户接口分开，用一个头文件描述数据和访问数据的接口；
- 使用命令行来驱动的用户接口，不再使用curses库，目的是把精力集中到数据接口的实现；

<!-- more -->

## 源代码行数sloc

- C&curses：554
     该例程“使用curses函数函数库管理基于文本的屏幕”，即使用了非常多的curses库函数，管理屏幕输入/出、键盘输入、窗口/子窗口等。使用细节暂且不深究。
     学习该例程的目的是，厘清使用C语言完成该程序的逻辑流程，及，与Shell的区别。
-  Shell：299


## 实现对比

`TITLE_FILE` 存储标题信息 ： 目录编号,标题,曲目类型,艺术家
`TRACKS_FILE` 存储曲目信息： 目录编号,曲目编号,曲名
目录编号因为是惟一的，所以用来关联2个文件，标题文件的一个数据项一般会对应曲目文件的多行数据。
 
### “增”操作
 
- C&curses：仅指向标题文件中增加一条CD记录。而CD的曲目信息，通过“改”操作录入。
    `fopen(), wgetnstr(), fprintf(), fclose().`
- Shell：读取标准输入，然后重定向到标题文件，这样就增加了一条CD记录；然后通过同样的方式增加曲目信息。
    `read, >>`
 
### “查”操作
 
- C&curses：引导用户输入需要查找的标题，逐行读取标题文件，进行匹配。
    不会把所有CD以列表形式呈现，所以针对某张CD操作时，必须先“查”，目前版本仅支持“标题”查找。
    需要变量 current_cd 存储“当前CD标题”，“增、查”2个操作会更新该变量，“删”重置该变量为`'\0'`。
    `wgetnstr()（curses库函数）, fopen(), fgets(), strstr() or strcmp(), fclose().`
- Shell：获取用户需要查找的信息，grep查找并重定向到临时文件；读取匹配的第一行，以获取CD目录编号等信息； 再次调用grep查找与目录编号匹配的曲目。
    `read, grep, wc -l, cut`
 
### “删”操作
 
- C&curses：删除当前CD记录，及其所有曲目。
    由于曲目文件每个条目，也就是每行的开头是目录编号；查找曲目文件中与“当前目录编号”不匹配的行，并写入临时文件，而匹配的行被丢弃，以达到删除的目的。
    `fopen(), fgets(), strncmp(), fputs(), fclose().`
- Shell：调用grep反向查找
    `grep -v, mv`
 
### “改”操作
 
- C&curses, Shell：不支持针对某个曲目编号的修改，需要先删除某个目录编号的所有曲目，然后重新录入。
    与“删api、增api”相同。


## task2 done 2014-07-23

完成6.10节[使用curses函数函数库（c语言）完成CD数据库应用程序](https://github.com/li2/beginning-linux-programming-exercise/blob/master/mini_cd_manager.c)的练习。
   
一个管理CD唱片的程序[《linux程序设计第4版》6.10节例程]。 使用菜单完成“增、删、改、查”四项功能，使用文本存储数据。


## task1 done 2014-07-21

完成2.8节[使用shell完成CD数据库应用程序]( https://github.com/li2/beginning-linux-programming-exercise/blob/master/mini_cd_manager.sh )的练习。
 
- 把时间推向两年前，学习《Linux程序设计》2.8的例程时，很吃力；
- 去年一年命令行为日常工作所用；
- 今年五月份通读了《shell脚本学习指南》；
- 今年七月份再看2.8节，涉及的模式匹配运算、算数展开运算、重定向、命令行参数、test命令、IFS、grep、cut、read等，无障碍，可快速理解程序。
 
**结论**
 
- 如果学习起来很吃力而产生了“怎么这么笨”的负面情绪时，必须暂时放下，而原因必然是缺乏相关主题的知识；
- 在有成效地通读某个主题的专业书籍之后，就具备了某种印象，在遇到与主题相关的代码段时，就可以快速理解其含义，并且加深对该主题的理解；
- 多加练习。
 
  
## Linux程序设计第4版 学习计划
 
**2014-07-20**
![计划](/assets/img/unix/begin-linux-programming-学习计划_20140720.jpg)
![ 每天使用shell来熟悉这些基本功能 ](/assets/img/unix/shell脚本学习指南-表6-2-每天使用.png)

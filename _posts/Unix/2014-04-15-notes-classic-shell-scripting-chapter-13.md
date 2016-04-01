---
layout: post
title: 「Shell脚本学习指南笔记」第13章-进程
filename: 2014-04-15-notes-Classic-Shell-Scripting-chapter-13.md
category: Unix
tags: [shell]
---

进程（process）是执行中程序的一个实例（instance）。涉及的概念：时间片段（time slice）、文本切换（context switches）、调度器（scheduler）、平均负载（load average）。详见操作系统类书籍。
本章只说明如何建立进程、列出进程、删除进程、传递信号给进程、监控进程。P363
  
<!-- more --> 

[英文版下载地址](http://files.cosmicduck.net/public_uploads/Classic_Shell_Scripting.pdf)
 
 
## 进程建立

------

很多程序由shell启动：每个命令行里的第一个单词是识别要执行的程序。
 
 
## 进程列表

------

进程状态（process status）命令：ps。`ps -efl`以SystemV形式显示冗长信息；`ps aux`以BSD形式显示冗长信息。
命令top可以动态观察进程状态。
 
 
## 进程控制与删除

------

kill命令可以终止进程。通过传递信号（signal）给指定的进程，进程收到信号后，处理或忽略。
只有进程的拥有者、root、内核、进程本身，可以传送递信号给它。
`kill -l` 列出支持的信号名称。
 
**删除进程的四个信号**
ABRT（中断），HUP（搁置），KILL，TERM（终结）。TODO
 
**捕捉进程信号**
`man -a signal` 查看所有关于信号的手册页

 
## 进程系统调用的追踪

------
 
 
## 进程帐

------
 
 
## 延迟的进程调度

------

有4种方法可以延迟进程，直到设定的时间才执行。
 
**sleep 休眠**
sleep会使调用它的进程休眠，直至计时满时被调度器唤醒。只占用很少的资源。
 
**at 在指定时间执行**
 
```bash
# 在指定的时间，执行command-file里的命令
# atq命令列出队列里的所有工作，atrm删除
at 21:00             < command-file  在21:00执行
at now               < command-file  立刻执行
at now + 10 minutes  < command-file  10分钟后执行
at now + 10 hours    < command-file  10小时后执行
at 0400 tomorrow     < command-file  明天04:00执行
at 4 June            < command-file  6月4日执行
at teatime           < command-file  下午茶时间（16:00）执行
```
 
**batch 采取批处理模式**
 
**crontab 重复执行计划任务**
 
```bash
$ crontab -l
# 列出目前的corntab调度
# mm      hh      dd      mon     weekday         /path/to/command
# 00-59   00-23   01-31   01-12   0-6(0=Sunday)   /path/to/ command            
  15      *       *       *       *               每小时的第15分钟执行
  */15    *       *       *       *               每15分钟执行
  0       6       *       *       1               每周1的06:00点执行
  0       8-17    *       *       0,6             每周末的08:00至17:00间，每小时执行一次
  55      23      *       *       *               每天的23:55执行
```
 
crontab会把command中的%更换为换行符，加上反斜杠可以避免：
 
```bash
`date +\%Y.\%m.\%d`
```
 
crontab文件里的命令路径设置严格，通常仅/usr/bin，所以安全的做法是写出完整路径名称：
 
```bash
/usr/local/bin/your_command_file
PATH=/usr/local/bin:$PATH your_command_file
``` 
 
 
## /proc文件系统

------


## 版本

li2 于上海闸北 
2014-04-15 ~ 2014-05-15, v1

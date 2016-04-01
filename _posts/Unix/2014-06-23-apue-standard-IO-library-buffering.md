---
layout: post
title: 「UNIX环境高级编程笔记」- 标准IO库 - 缓冲
filename: 2014-06-23-Standard-IO-Library-Buffering.md
category: Unix
tags: []
---

缓冲 [5.4]
标准I/O库提供缓冲的目的是减少read和write调用次数，节省执行I/O所需的CPU时间。包括3种：
 
- 无缓冲：立刻执行I/O. -------------------标准错误流
- 行缓冲：遇到换行符执行I/O. -------------连接到终端设备的其它流
- 全缓冲：填满标准I/O缓冲区后执行I/O. ----除以上两种情况外
 
<!-- more -->

冲洗（flush）：标准I/O缓冲区的写操作。[5.4 缓冲]
缓冲区可由标准I/O例程自动冲洗（例如当填满一个缓冲区时），或者调用`int fflush(FILE *fp)`强制冲洗。
在UNIX环境中，flush有两种意思。在标准I/O库方面指誊写到磁盘文件；在终端驱动方面（比如tcflush）指丢弃。
 
调用 `#include <stdio.h>  int fclose(FILE *fp)` 关闭一个打开的流。在文件被关闭之前，冲洗缓冲的输出数据，丢弃缓冲的输入数据。[5.5 打开流]
当一个进程正常终止时（调用exit(), 或从main返回），
exit()执行标准I/O库的清理关闭操作：为所有打开流调用fclose()。[7.3 进程终止]

---
layout: post
title: Linux设备驱动程序开发
category: 嵌入式
tags: [linux-device-drivers]
---

## 第 1 部分：字符设备驱动程序 Character Device Drivers

问题：

- 应用程序如何与字符设备驱动程序交互？比如，用户空间的open是如何一步一步调用驱动程序的open? 
- 当完成一个设备的驱动程序后，必须调用API注册到kernel。问题是kernel提供了什么？做了什么？使得交互成为可能。
 
理解上述问题需要具备如下知识：

- [I/O 体系结构（设备文件的标准I/O操作）  I/O Architecture (Standard Operations for Character Devices)](http://li2.me/ldd/IO-Architecture-and-Cdev-Standard-Operations/)
- [字符设备文件 & 设备文件节点的生成及打开 Character Device Files and Inodes](http://li2.me/ldd/Character-Device-Files-and-Inodes/)
 
在此基础上，将围绕[“创建并注册一个新的字符设备”](http://li2.me/ldd/Create-and-Register-a-New-Character-Device-Driver/)展开如下论述：

- 如何实现 how to create a new character device driver;
- 如何注册 how to register character device driver;
- 幕后的内核机制 mechanism in kernel.


## 第 2 部分：串行设备驱动程序(TTY、UART、Line Disciplin) Serial Device Drivers

[TTY设备驱动程序](http://li2.me/ldd/TTY-Device-Drivers/)的内核数据结构定义 struct tty_driver, 开发针对特定设备的 TTY 驱动程序，仅2步：

- 定义 tty_driver, 关键是完成操作硬件设备的入口函数集 struct tty_operations;
- 注册 tty_driver, 通过调用 TTY I/O Core(内核为支持 tty driver 而定义的一系列数据结构和函数) API.

完成上述2个步骤使得用户空间可通过设备文件访问硬件，使用设备驱动程序提供的各种操作[ ELDD-6.3, LDD3-18 ]. 

TODO


## 第 3 部分：I2C设备驱动程序
TODO


## 第 4 部分：通用驱动程序模型 The Generic Driver Model
TODO

## 第 5 部分：参考书目

- LKA  --  《深入linux内核结构 Professional Linux Kernel Architecture》Wolfgang著，郭旭译，人民邮电出版社201006.
讨论底层基础设施的实现。

- ELDD  --  《精通Linux设备驱动程序开发 Essential Linux Device Drivers》宋宝华译，人民邮电出版社201006.

- ILDD  --  《深入Linux设备驱动程序内核机制 Internals of Linux Device Drivers》陈学松著，电子工业出版社201201.
强烈推荐 ILDD，作者并没有大量复制内核代码占据篇幅，而是以问题为导向，从代码中提取关键部分，通过关键数据结构和代码执行流程，条理清晰地阐述某个功能的实现。作者画的流程图使问题更容易理解。（书中2.7节解决了笔者的一个问题后，有上述感悟）

- LDD3  --  《Linux设备驱动程序第3版 Linux Device Drivers》魏永明译，中国电力出版社200601.
使用例程创建新的驱动程序。


## 编辑历史
- 2013-12-18 初稿，完成“字符设备驱动程序”笔记整理 by li2 上海闸北
- 2014-01-05 使用markdown编辑格式

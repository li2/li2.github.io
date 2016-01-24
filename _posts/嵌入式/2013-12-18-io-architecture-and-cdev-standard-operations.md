---
layout: post
title: I/O体系结构（设备文件的标准I/O操作）
category: 嵌入式
tags: [linux-device-drivers]
---

> 按照经典的UNIX箴言“万物皆文件”（everything is file），对外设的访问可利用/dev目录下的设备文件完成，程序对设备的处理完全类似于常规文件。设备驱动程序的任务是支持应用程序经由设备文件与设备通信，即，按适当的方式在设备上读取/写入数据。[ LKA-1.3.8 设备驱动程序、块设备和字符设备 ]
 
> 与外设的通信称为输入输出，缩写为I/O. 在实现外设I/O时，内核必须处理3个可能出现的问题：[ LKA-6.1 ]

> - 硬件寻址；
> - 向应用程序提供访问设备的方法，应当采取统一的方案，确保应用程序能够在不考虑硬件设备的情况下进行互操作；
> - 用户空间需要知道内核中有哪些设备可用。

> 这些问题通过层次化的多个抽象层解决。层次结构的最底层是硬件设备，它通过总线系统连接到系统CPU和其它设备。如下图 [ LKA-6.1-图6-1 ]：
> 
> ![LKA-6.1-Figure6.1-Layer-model-for-addressing-peripherals](/assets/img/ldd/LKA-6.1-Figure6.1-Layer-model-for-addressing-peripherals.png)

<!-- more -->

## 文件I/O的标准库定义

应用程序调用open打开设备文件，若成功则返回文件描述符。读或写一个设备文件时，使用open返回的文件描述符标识该文件，将其作为参数传递给read或write. [ APUE-3 ]

    int open(const char *pahtname, int oflag, .../* mode_t mode */);
    ssize_t read(int filedes, void *buf, size_t nbytes);
    ssize_t write(int filedes, const void *buf, size_t nbytes);
    int close(int filedes);


## 文件I/O的内核定义
 
内核（kernel/include/linux/fs.h）定义了一组文件操作的函数指针集合，包括打开、读取、写入等。 [ LKA-8.3.4, 8.2.4 ], [ LDD3-3.一些重要的数据结构.文件操作 ]

    struct file_operations {
        int (*open) (struct inode *, struct file *);
        ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
        ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *); 
    }


## 编辑历史

- 2013-12-18 初稿
- 2014-01-06 使用markdown编辑格式

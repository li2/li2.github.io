---
layout: post
title: 创建并注册一个新的字符设备驱动程序
category: 嵌入式
tags: [linux-device-drivers]
---

## 字符设备的内核数据结构定义：struct cdev

字符设备由 cdev 表示。[ LKA-6.4.1 ]  同时，内核维护了一个数据库[ LKA-6.2.5-设备数据库 ]，记录所有注册到内核的“活动的“ cdev 实例。

    struct cdev {
        struct kobject kobj;          //嵌在struct cdev 中的内核对象，用于管理该数据结构
        struct module *owner;         //指向提供驱动程序的模块
        struct file_operations *ops;  //文件操作
        struct list_head list;        //实现链表，链接所有指向该设备的inode
        dev_t dev;                    //设备号
        unsigned int count;           //与该设备关联的从设备的数目
    };
可以将 cdev 嵌入到某个特定的字符设备数据结构中，以实现特定硬件设备的功能。
 
 
## 字符设备注册到内核的过程

<!-- more -->

kernel/fs/char_dev.c
步骤：

1. 注册或申请设备号 [ LDD3-3.主设备号和次设备号 ]：
主设备号若为0则动态分配 alloc_chrdev_region(); 否则静态分配 register_chrdev_region();
2. 调用 cdev_alloc() 动态分配 cdev 内存：
3. 调用 cdev_init() 实例化 cdev，关联 file_operations 与 cdev：
4. 调用 cdev_add() 关联设备号与 cdev，调用 kobj_map(cdev_map, )把 cdev 添加到内核设备数据库 cdev_map，返回后该 cdev 处于活动状态，所谓活动状态，是指可以调用 kobj_lookup(cdev_map,设备号,...)获取，并使用它的成员。
上述几个步骤也可以通过调用一个函数完成：kernel/include/linux/fs.h/register_chrdev(major, name, fops);
5. 创建设备文件：参考“设备文件节点的生成”。
 
 
## 字符设备驱动程序的实例代码 

- ILDD-2.1，实现了一个简单的字符设备驱动程序的内核模块，通过insmod加载到内核，通过mknod创建设备文件节点，然后编写一个小的应用程序，调用字符设备驱动提供的服务。
这个例子展示了字符设备驱动的典型架构，并演示了用户空间与内核驱动空间的交互。

- ELDD-5.2-设备实例：系统CMOS。演示了如何把 struct cdev 嵌入到（特定设备的）自定义数据结构 struct cmos_dev 中（而非指针引用），以针对特定设备实现更复杂的功能。
注册由初始化函数 cmos_init() 完成，并且动态分配了 cmos_dev 的存储空间。
打开设备时执行 cmos_open()，正如在“字符设备文件的打开，步骤（2）”中提到的，驱动已经获得了该字符设备（inode->i_cdev），因此 cmos_open()可以通过宏 container_of 获得包含 cdev 的 cmos_dev.
container_of: cast a member of a structure out to the containing structure, 从结构的成员来获得包含该成员的结构的实例。[ LKA-C.2.7 ]
 
- 与此类似的例子是，LDD3，全章内容围绕字符设备驱动程序scull展开，实例和理论概念穿插讲解；scull 驱动程序和硬件无关，仅是操作内核分配的一些内存，因此可移植。但是为 scull 设计的字符设备数据结构略复杂。

 
## 编辑历史

- 2013-12-18 初稿 by li2 上海闸北
- 2014-01-07 使用markdown编辑格式

---
layout: post
title: 字符设备文件 & 设备文件节点的生成及打开
category: 嵌入式
tags: [linux-device-drivers]
---

## 设备文件节点的内核数据结构定义：struct inode

虚拟文件系统中的每个文件都关联到一个inode，用于管理文件的属性。对单个文件，可能会有多个表示文件描述符的 struct file，但它们都指向惟一的 struct inode. 用户空间打开设备文件时，内核会寻找关联的 inode, 利用其中记录的设备号等关键信息完成后续的操作。 [ LDD3-3.一些重要的数据结构.inode结构 ], [ LKA-6.3.1 ]
struct inode 包含了大量与文件有关的信息，而与设备驱动程序有关的成员：

    kernel/include/linux/fs.h
    struct inode {
        dev_t i_rdev;                   //真正的设备号
        struct file_operations *i_fop;  //被赋予通用的设备文件操作函数集
        union {
            struct cdev *i_cdev;        //指向特定的字符设备结构的指针
        };
    }
 

## 设备文件节点的生成
 

在Linux系统下，设备文件是特殊的文件类型，它提供的机制使得，用户空间可以使用驱动程序定义的接口。如果驱动程序只为内核中的其它模块提供服务，则没有必要生成设备文件。
设备文件的创建依赖于“设备文件名”和“设备号”，有2种方法：

<!-- more -->

- 设备节点的静态生成 [ ILDD-2.6 ]
使用linux系统命令：

        mknod [-m MODE] NAME TYPE MAJOR MINOR
        # mknod /dev/demodev c 2 0
        # ls -l /dev/demodev
        crw-rw-rw- root     root       2,   0 2013-12-10 06:26 demodev

- 设备节点的动态生成
class_create(owner, name);
kernel/drivers/base/core.c/device_create();
调用device_create()向系统添加设备，属于“在sysfs文件系统中建立系统硬件拓扑关系结构图“四种情况中的后两种。[ ILDD-9.4-P354 ], [ ILDD-9.3.4-P342]
device_create()产生uevent， udevd监听uevent后根据相关规则创建设备节点。 [ ELDD-5.2.1-P86], [ ELDD-4.3.1 udev ]
 
以上2种方法之一，会生成2个“对象”：

- /dev目录下的设备文件 demodev;
- struct inode 对象，用于在内核空间表示设备文件；

/dev/demodev 与新生成的 inode 之间的关联由文件系统管理；设备文件节点的生成涉及VFS和特定文件系统的技术细节，从驱动程序开发的角度，只需关注与驱动程序相关联的部分。
inode对象被创建后，会被初始化： [ ILDD-2.6 ], [ LKA-6.3.2 ]

    kernel/fs/inode.c
    void init_special_inode(struct inode *inode, umode_t mode, dev_t rdev)
        inode->i_mode = mode;
        if (S_ISCHR(mode)) {
            inode->i_fop = &def_chr_fops; //字符设备的inode被赋予字符设备通用操作
            inode->i_rdev = rdev;
    }
 
至此，字符设备文件的 struct inode 已经具有如下数据：

- 设备号；
- 指向通用的字符设备文件操作函数集struct file_operations def_chr_fops (kernel/fs/char_dev.c).
def_chr_fops主要定义了 .open = chrdev_open；当用户空间调用open(“/dev/demodev”, …)时，chrdev_open() 会被调用，它根据inode中记录的设备号查找 struct cdev（由对应的字符设备驱动程序向内核注册），然后用户空间就可以调用cdev中的file_operations，从而控制设备。详见下述“字符设备文件的打开“。
 
 
## 字符设备文件的打开

用户空间调用open后，引发的系统调用所做的事情：
步骤[ ILDD-2.7 ]：
 
>1. 通过系统调用 sys_open() 进入内核空间；
    1. 在内核空间中主要由 kernel/fs/open.c/do_sys_open 发起设备文件的打开操作，首先获取设备文件 /dev/demodev 的 inode （根据设备文件名查找节点，如同设备文件节点的生成，也涉及文件系统的技术细节，暂且不考）；
    1. 然后调用 inode 的open函数 inode->i_fop->open()，对于字符设备而言，实际执行的是 chrdev_open()，（参考“设备文件节点的生成”一节中对字符设备 inode 的分析）；
    1. 每次打开都会创建一个未使用的文件描述符fd，一个新的struct file对象 filp，用于跟踪这次操作；内核进程会维护一个文件描述符表，fd 作为索引值，把 filp 赋值给该表，用于关联 filp 和 fd；
>
>            struct file 中与设备驱动相关的成员：
            struct file_operations    *f_op; 文件操作函数集接口；
            atomic_long_t f_count; file的使用计数；
            unsigned int f_flags; 设备文件的打开模式；
            void *private_data; 记录设备驱动程序的自定义数据，方便数据传递；
>
>2. chrdev_open() 通过 inode中记录的设备号（inode->i_rdev）在字符设备数据库（cdev_map）中查找对应的字符设备 cdev（参考“字符设备的注册”）；
>2. 查找成功后，把 inode的 i_cdev 成员指向该cdev，这样再次打开该设备文件节点时，可以通过成员i_cdev 直接使用该cdev，而不必查找 cdev_map（有些书中称之为“缓存”）；
>2. 把 cdev 的 file_operations 与 filp 关联（filp->f_op = inode->i_cdev->ops），并执行 filp->f_op->open(), 即 cdev 的 open 函数；
>2. 将 fd 返回到用户空间。
 
至此，用户的 open 终于调用了驱动的 open，并且后续的 read、write、ioctl 等函数的调用，可以通过 fd 获取设备文件对应的 filp，进而调用 f_op 中实现的设备驱动函数。
结合关键数据结构，观察大体的执行流程图 [ ILDD-2.7-图2-10 ], [ LKA-6.4.2-图6-7 ]
《ILDD 图2-10》非常经典地展示了设备文件打开的代码执行流程、内核数据结构之间的关系。 

> 
> ![ILDD-2.7-Figure2.10- Flowchart of opening character device node](/assets/img/ldd/ILDD-2.7-Figure2.10-Flowchart-of-opening-character-device-node.png)

>
> ![LKA-6.4.2-Figure6.7- Relations between data structures for the representation character devices](/assets/img/ldd/LKA-6.4.2-Figure6.7-Relations-between-data-structures-for-the-representation-character-devices.png)

chrdev_open()是用于字符设备的标准（或通用的或基本的）打开操作，在调用顺序上，它处于用户open和特定设备驱动open之间。这种“分层”应该是考虑了字符设备的多样性，每个特定的设备都需要一组独立、自定义的操作函数。正是 chrdev_open() 为用户空间提供了打开字符设备的一致方式。 [ LKA-6.3.3 ], [ LKA-6.4.2 ]


## 字符设备文件的关闭

用户空间的 close 最终会执行 file->f_op->release(inode, file), 即 cdev 的 release 函数。[ ILDD-2.7-P83]


## 编辑历史
- 2013-12-18  初稿 by li2 上海闸北
- 2014-01-07  使用markdown编辑格式

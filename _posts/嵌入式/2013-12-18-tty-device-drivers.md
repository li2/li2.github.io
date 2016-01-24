---
layout: post
title: TTY设备驱动程序
category: 嵌入式
tags: [linux-device-drivers]
---

TTY 设备驱动程序的内核数据结构定义 struct tty_driver, 开发针对特定设备的 TTY 驱动程序，仅2步：

- 定义 tty_driver, 关键是完成操作硬件设备的入口函数集 struct tty_operations;
- 注册 tty_driver, 通过调用 TTY I/O Core(内核为支持 tty driver 而定义的一系列数据结构和函数) API.

完成上述2个步骤使得用户空间可通过设备文件访问硬件，使用设备驱动程序提供的各种操作[ ELDD-6.3, LDD3-18 ]. 驱动开发者不仅需要会调用这些 API, 而且需要了解 API 的内部实现。本文关注驱动的注册过程，设备的打开、读、写的细节。 总的来说：

- 驱动内嵌了字符设备，因此复用了字符设备文件的生成、查找、打开机制；
- 注册时添加驱动到 TTY I/O Core 管理的链表中；
- 一个驱动可以管理多个设备，每个设备拥有惟一的设备号；
- 设备对用户空间来讲是设备文件节点，对内核空间来讲是 struct tty_struct; 注册时为每个设备创建设备文件节点（也可在注册设备驱动之后、打开设备之前创建）；首次打开某个设备时由 TTY I/O Core 创建 tty_struct 用以实例化该设备；
- 设备号非常重要，打开设备时，通过设备号获取驱动和索引号（该设备是驱动管理的第几个设备）；得到驱动后通过索引号查找该驱动管理的设备实例 tty_struct;
- TODO 线路规程

<!-- more -->

## tty_driver 的注册过程

注册 TTY 设备驱动的流程图，以及，表示 TTY 设备驱动各个数据结构之间的关系：


![weiyi-LDD-Figure-tty register](/assets/img/ldd/weiyi-LDD-Figure-tty-register.png){:.foo}

1. 动态分配 tty_driver, 把支持的 tty 设备数量写入 .num, 由 alloc_tty_driver()完成；
2. 初始化 tty_driver: 
    - .driver_name, 与.name的区别参考 LDD3-18.小型TTY驱动程序.P543;
    - .name, 与次设备号组合成设备文件名称;
    - .major, 主设备号，若为0则注册时动态分配；
    - .type, 有3种类型的 tty 驱动程序(console, serial, pty), 一般设置新的驱动为 TTY_DRIVER_TYPE_SERIAL, [ LDD3-18-P539 ]; 
    - .flags, 标志位，一般设置为 TTY_DRIVER_REAL_RAW;
    - .init_termios, 终端属性初始化值；
    - .ops, 由驱动开发者完成的 struct tty_operations 类型操作函数集，需要完成 .open, .close, .write;
    - 其它成员 .cdev, .ttys, .termios, .tty_drivers 将由 tty_register_driver() 完成初始化。

3. 调用 int tty_register_driver(struct tty_driver *driver) 向 TTY I/O Core 注册该设备驱动程序：
    - 分配设备号;
    - 注册字符设备（[回顾“创建并注册一个新的字符设备”](li2.me/ldd/Create-and-Register-a-New-Character-Device-Driver/)），它的文件操作函数集 kernel/drivers/tty/tty_io.c/struct file_operations tty_fops 是 TTY I/O Core 为支持 tty_driver 而定义；
    - 添加 .tty_drivers 到 TTY I/O Core 管理的链表，该链表用于管理注册到 TTY I/O Core 的所有 tty_driver, 通过调用 get_tty_driver() 以设备号为索引遍历链表获取，后续分析 open 过程时会有详细说明（链表相关知识参考 [ LKA-1.3.13 ], [ LDD3-11.链表]）；
    - 调用 tty_register_device(), 根据分配的设备号和 .name 生成 .num 个设备文件;
    - 初始化设备文件的 inode; 这两步建立了用户空间与内核空间交互的文件接口（[回顾“字符设备文件 & 设备文件节点的生成及打开”](li2.me/ldd/Character-Device-Files-and-Inodes/)）；
    - 写与驱动程序相关的信息到文件 /proc/tty/drivers;

至此，tty_driver 注册已经完成，它所有的初始化工作都是为后续的文件操作做准备。驱动开发者几乎只需实现并注册设备驱动, 其它工作由 TTY I/O Core 托管，后续将从 open, close, read, write 分析实现的细节。



## TTY 设备的打开操作

打开 TTY 设备的流程图，以及，表示 TTY 设备各个数据结构之间的关系：


![weiyi-LDD-Figure-tty open](/assets/img/ldd/weiyi-LDD-Figure-tty-open.png){:.foo}

1. TTY 设备驱动基于字符设备构建，首先会执行字符设备的打开操作，将执行 filp->f_op->open(inode,filp) 即 tty_open(), Linux 中基于 cdev 创建的设备，打开操作与此相同；
2. tty_open()是 TTY 设备的标准打开函数，由它进一步调用驱动开发者定义的 open 入口函数：
    - 查找 tty_driver: 以设备号为索引遍历链表获取 tty_driver 和 设备的索引号 index（若 driver 管理两个设备，打开第一个设备返回0，打开第二个设备返回1，index 被用于 tty_struct 索引）；如果没有与设备号匹配的驱动，返回 ENODEV "No such device", 打开失败；
    - 查找 tty_struct: 以索引号在 tty_driver 中查找，若有，说明设备被打开过，执行 tty_reopen()更新打开次数；若无，说明首次打开，需要建立设备的 tty_struct;
        1. 动态分配 tty_struct 的存储空间，然后初始化它，包括：
            - 指向一个动态分配的线路规程 struct tty_ldisc, 获取默认的线路规程 N_TTY 的操作函数集(drivers/tty/n_tty.c/struct tty_ldisc_ops tty_ldisc_N_TTY);
            - 初始化一个链表头，该链表用于管理多次打开设备时生成的 struct file;
            - 指向 tty_driver;
            - 指向 tty_driver.ops;
            - 记录设备的索引号 index;
            - 记录设备文件的名称；
        2.  设置 tty_driver, tty_struct 的终端属性；打开次数 tty_struct.count 加1; 被 tty_driver 指向；
        3.  执行线路规程的 open 操作，为线路规程分配数据读缓冲区；
    - 通过之前初始化的链表记录每一次打开设备产生的 struct file [回顾这里](li2.me/ldd/Character-Device-Files-and-Inodes/), 并建立两者的双向关联，所以，后续用户空间的操作就可以通过 file 获取 tty_struct, file_tty();
    - 执行注册设备驱动时定义的 open();
3. 将 fd 返回到用户空间。

至此，tty 设备的打开操作已经完成，用户空间的 open 最终调用到了底层驱动的 open.
第一阶段的打开工作完全遵循字符设备的打开操作，因为 tty 设备建立在字符设备基础之上；第二阶段的打开工作由 tty_open()完成，当首次打开设备时创建 tty_struct, 这个数据结构由 TTY I/O Core 创建并管理，它一面连接内核空间的驱动，另一面连接用户空间的 file.  后续的 read、write、ioctl 等操作，通过 fd 获取设备文件对应的 file, 从而获得设备驱动的文件操作入口函数集，并执行：

    tty 设备的关闭操作：filp->f_op->release(inode,filp), 即 tty_release();
    tty 设备的写操作：file->f_op->write(file, buf, count, pos), 即 tty_write();
    tty 设备的读操作：file->f_op->read(file, buf, count, pos), 即 tty_read();
    相关的系统调用位于 ./kernel/fs/read_write.c/vfs_read(), vfs_write() ./kernel/fs/open.c/filp_close().



##  struct tty_operations

    //由 tty_driver_lookup_tty()调用
    struct tty_struct * (*lookup)(struct tty_driver *driver, struct inode *inode, int idx);
    //由 tty_driver_install_tty()调用
    int  (*install)(struct tty_driver *driver, struct tty_struct *tty); 
    //由 tty_driver_remove_tty()调用
    void (*remove)(struct tty_driver *driver, struct tty_struct *tty);
    //以上3个函数指针由 tty_io.c 内部使用，用于查找、赋值、删除 struct tty_drive 内部的 struct tty_struct 列表；如果函数未实现(NULL)，则默认使用 ttys array. 

    //由 tty_open()调用
    int  (*open)(struct tty_struct * tty, struct file * filp);
    //由 tty_release()调用
    void (*close)(struct tty_struct * tty, struct file * filp);
    //由 tty_write()调用
    int  (*write)(struct tty_struct * tty, const unsigned char *buf, int count);
    //以上3个函数指针在打开、关闭、写 tty device 时被调用，必须实现。

    //其它 TODO


## TTY 的线路规程

> 线路规程提供的机制，使得用户运行不同的应用程序时，能够使用相同的串行驱动程序。底层的物理驱动程序和 tty 驱动程序负责从硬件上接收数据，而线路规程负责处理这些数据，并在内核空间和用户空间之间传递数据 [ ELDD-6.4 ]。
> 串行子系统支持17种标准的线路规程。打开串行端口时 TTY I/O Core 默认绑定的是 N_TTY, 它实现了终端 I/O 处理。

### 正在使用的线路规程
    
    $ cat /proc/tty/ldiscs
    n_tty    0 (线路规程名，线路规程ID号)
    input    2

### 设置线路规程 TODO

    /kernel/Documentation/serial/tty.txt:Line disciplines are registered with tty_register_ldisc()
    kernel/drivers/misc/ti-st/st_core.c:    err = tty_register_ldisc(N_TI_WL, &st_ldisc_ops);
 

## TTY 设备的写操作

    tty_write() -> do_tty_write() -> ld->ops->write() == n_tty_write() -> tty->ops->write() [ELDD-6.4.3]
    static ssize_t n_tty_write(struct tty_struct *tty, struct file *file, const unsigned char *buf, size_t nr)
        while (1) {
            //O_OPOST(tty)=0, tty->flags=0A00, if(?=0)
            //问题是，按代码分析O_OPOST(tty)应该为1，可以实际的打印值为0? TODO
            if (O_OPOST(tty) && !(test_bit(TTY_HW_COOK_OUT, &tty->flags))) {
                while (nr > 0) {
                    //这个函数最终调用的也是tty->ops->write();
                    ssize_t num = process_output_block(tty, b, nr);  
            } else {
                //从打印结果来看，写操作走到这个分支
                while (nr > 0) {
                    c = tty->ops->write(tty, b, nr);

关于 TTY_HW_COOK_OUT TODO

    ./kernel/include/linux/tty.h:#define TTY_HW_COOK_OUT    14      /* Hardware can do output cooking */
    ./kernel/drivers/tty/n_tty.c:                       !(test_bit(TTY_HW_COOK_OUT, &tty->flags))) {
    ./kernel/drivers/tty/n_tty.c:           if (O_OPOST(tty) && !(test_bit(TTY_HW_COOK_OUT, &tty->flags))) {
    ./kernel/drivers/char/sx.c:             set_bit(TTY_HW_COOK_OUT, &port->gs.port.tty->flags);
    ./kernel/drivers/char/sx.c:             clear_bit(TTY_HW_COOK_OUT, &port->gs.port.tty->flags);


## TTY 设备的读操作

struct tty_operations{}中没有read函数。
对于中断驱动的设备，读取数据的过程通常由一前一后两个线程组成 [ELDD-6.4.2]。

1. 用户空间读数据发起的顶层线程：
用户空间的 read 把数据搬运到用户空间的缓冲区 [LDD3-3-图3.2]。

        tty_read() -> ld->ops->read() == n_tty_read() -> tty->ops-> read() 
        static ssize_t n_tty_read()
            add_wait_queue(&tty->read_wait, &wait);
            while (nr) {
                //tty->icanon=0, L_EXTPROC(tty)=0? TODO
                if (tty->icanon && !L_EXTPROC(tty)) {
                } else {
                    uncopied = copy_from_read_buf(tty, &b, &nr);
                }
            }
        static int copy_from_read_buf()
            retval = copy_to_user(*b, &tty->read_buf[tty->read_tail], n);

至此，可以看出 read 操作最终是从 tty_struct.read_buf 中读取数据，问题是，谁向其中写入了数据？

2. 接收硬件数据的 ISR 唤醒的底层线程：
考虑到串行设备驱动程序层次结构的划分，

        kernel/drivers/tty/tty_buffer.c
        void tty_flip_buffer_push(struct tty_struct *tty)
            //TODO tty->low_latency=1? tmc_timer()中赋值为1
            if (tty->low_latency)
                flush_to_ldisc(&tty->buf.work.work); 
       
        static void flush_to_ldisc(struct work_struct *work)
                while ((head = tty->buf.head) != NULL) {
                    disc->ops->receive_buf(tty, char_buf, flag_buf, count);

        static void n_tty_receive_buf()
            //tty->real_raw=1. TODO
            if (tty->real_raw)
                memcpy(tty->read_buf + tty->read_head, cp, i);


    - 底层驱动程序（比如 UART Device Drivers serial_omap_irq()）会把硬件接收到的数据“推送”到交替缓冲区 tty->buf（驱动开发者不必为每个 tty 驱动实现自己的缓冲区逻辑），通过调用 tty_insert_flip_char()完成;
    - 再推向线路规程，必需调用 tty_flip_buffer_push(); 设置 tty->low_latency 标志位会调用  flush_to_ldisc()使得数据“推送”立刻执行，否则放入队列排队 [ LDD3-18.怎么没有read函数-P550], [ ELDD-6.4.2 ]；
    - “推送”工作交由线路规程的文件操作函数 tty_ldisc_ops->receive_buf()完成，而 N_TTY 把数据从交替缓冲区 tty->read_head 拷贝到 tty->read_buf.

至此，读操作分析完毕。

TODO 
修正 [ ELDD-6.4-图6.7 ], 标注 tty_read()来自？
分析交替缓冲区数据收集的缓冲区(struct tty_bufhead, strcut tty_buffer), tty->flip.char_buf 这个是旧的数据结构接口 ELDD-6.3-P133


## TTY 设备的 ioctl 操作

> 当涉及检查特定于设备的功能和属性，超出了通用文件框架的限制时，只用输入输出命令很难完成。通过定义具有特殊含义的“魔术”字符串，并使用通用的读写函数，也可以完成此类任务。但如果要写入的数据和作为控制的“魔术”字符串相同，则会冲突。所以内核提供了 IOCTL 方法，每个设备驱动程序可以定义一个 ioctl 例程，使得控制数据的传输可以独立于实际的输入输出通道。标准库提供了 ioctl 函数，可以通过特殊的码值将 ioctl 命令发送到打开的文件 [ LKA-6.2.3 ]。

    long tty_ioctl()
        if (ld->ops->ioctl) 
            retval = ld->ops->ioctl(tty, file, cmd, arg);

记录 ioctl（set termios）流程 TODO
[tty_init_dev]1412, driver_name=pty_master,c_oflag=0, c_iflag=0, c_cflag=191, c_lflag=0, flags=2560.
[tty_init_dev]1412, driver_name=tmc_tty,   c_oflag=5, c_iflag=1280, c_cflag=3261, c_lflag=35387, flags=2560.

- 关于 tty->low_latency 
> 表示 tty 设备是否是个慢速设备，是否能接收高速传输的数据。tty 驱动程序可以设置该值。[ LDD3-18.tty_struct{}-P564 ]
> Indicates whether the tty device is a low-latency device, capable of receiving data at a very high rate of speed. The tty driver can set this value.
        
        kernel/drivers/tty/tty_buffer.c: if (tty->low_latency)
        kernel/include/linux/tty.h
        struct tty_struct {
            unsigned char low_latency:1; 

- 关于 tty->icanon
[Enable canonical mode (described below)](http://linux.die.net/man/3/termios)
        tty->icanon = (L_ICANON(tty) != 0);

- 关于 tty->real_raw
> tty 驱动程序和 tty 核心都使用 flags 变量表明当前驱动程序的状态以及该 tty 驱动程序的类型。这里定义了许多位的掩码宏操作，当对这些标志位进行测试和操作时，必须使用这些宏。驱动程序可以设置的 flags 变量中的三个位：TTY_DRIVER_REAL_RAW, 该标志表示 tty 驱动程序使用奇偶校验或者中断字符线路规程。这使得线路规程能以比较快的方式接收字符，因为它不必检查从 tty 驱动程序那里接收的每一个字符。由于速度提升的好处，所有的 tty 驱动程序通常都设置该位[ LDD3-18.termios结构-P544 ]。

        $ grep -r -w  --exclude-dir="*.svn" "TTY_DRIVER_REAL_RAW" kernel/drivers/tty/
        kernel/drivers/tty/n_tty.c:
        n_tty_set_termios()
            if (test_bit(TTY_HW_COOK_IN, &tty->flags)) {
                tty->real_raw = 1;
                return;
            }
            if (){
            } else {
                if (... (tty->driver->flags & TTY_DRIVER_REAL_RAW))
                    tty->real_raw = 1;
                else
                    tty->real_raw = 0;
            } 

- 关于 TTY_HW_COOK_IN
> 几乎与设置驱动程序 flags 变量为 TTY_DRIVER_REAL_RAW 的情况相同。该标志位通常不能由 tty 驱动程序设置[ LDD3-18.tty_struct-P563 ]。

        $ grep -r -w  --exclude-dir="*.svn" "TTY_HW_COOK_IN" kernel/drivers/
        整个driver目录，就只有3处，所以可以认定，在n_tty.c中，该标志位未被置位。
        kernel/drivers/tty/n_tty.c: if (test_bit(TTY_HW_COOK_IN, &tty->flags)) {
        kernel/drivers/char/sx.c:   clear_bit(TTY_HW_COOK_IN, &port->gs.port.tty->flags);
        kernel/drivers/char/sx.c:   set_bit(TTY_HW_COOK_IN, &port->gs.port.tty->flags);


## 查看 TTT 设备及设备驱动信息的系统命令

- 已注册的 TTY 驱动 

        # cat /proc/tty/drivers
        OMAP-SERIAL          /dev/ttyO     253 0-5 serial
        tmc_tty              /dev/tmc_tty  240 0-1 serial

- 已注册的 TTY 设备

        # ls /sys/class/tty | busybox grep ttyO
        ttyO0
        ttyO1
        ttyO2
        ttyO3

- 正在使用的线路规程 

        # cat /proc/tty/ldiscs
        n_tty       0
        input       2
        n_hci      15
        n_st       22


## 编辑历史

- 2013-12-23 初稿，完成文字部分整理
- 2014-01-05 完成2个流程图绘制
- 2014-01-11 使用markdown编辑格式

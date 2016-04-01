---
layout: post
title: 「UNIX环境高级编程笔记」- 进程控制 - 进程标识
category: Unix
tags: []
---

Process Control - Process Identifiers


每个进程都有一个惟一的进程ID。[8.2 进程标识符]
ID为0的进程通常是调度进程，又称交换进程（swapper）。是内核的一部分，又称系统进程。


ID为1的进程通常是init进程，在引导加载程序结束后由内核调用，读系统初始化文件并启动系统。
is invoked by the kernel at the end of the bootstrap procedure.
This process is responsible for bringing up a UNIX system after thekernel has been bootstrapped.
init进程不会终止。


> bootstrap 引导程序、自举。
> Boot Loader是在操作系统内核运行之前运行的一段小程序。
> 通过这段小程序，我们可以初始化硬件设备、建立内存空间的映射图，从而将系统的软硬件环境带到一个合适的状态，以便为最终调用操作系统内核准备好正确的环境。
> 参考[嵌入式BootLoader技术内幕 ](http://www.cnblogs.com/hnrainll/archive/2011/04/12/2013604.html)


ID为2的进程是页守护进程（pagedaemon）。此进程负责支持虚拟存储系统的分页操作。

```sh
weiyi:~$ ./a.out &
[1] 4723
weiyi:~$
getpid()  = 4723    //process ID of calling precess. （调用该API的）进程ID
getppid() = 4601    //parent process ID     父进程ID
getuid()  = 1000    //real user ID          实际用户ID
getgid()  = 1000    //real group ID         实际组ID
geteuid() = 1000    //effective user ID     有效用户ID
getegid() = 1000    //effective group ID    有效组ID
ps
PID TTY          TIME CMD
4601 pts/18   00:00:00 bash
4723 pts/18   00:00:00 a.out
4724 pts/18   00:00:00 ps

weiyi:~$ cat /etc/passwd | grep weiyi
weiyi    :x      :1000   :1000    :weiyi,,,           :/home/weiyi    :/bin/bash
username :passwd :userID :groupID :user personal name :home directory :login shell
[classic shell scripting 3.3.1 ]
```

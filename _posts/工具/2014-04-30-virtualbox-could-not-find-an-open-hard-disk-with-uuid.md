---
layout: post
title: 增加virtualbox的磁盘空间时遇到错误“Could not find an open hard disk with UUID”
category: 工具
tags: []
---

### 增加virtualbox的磁盘空间

[How do I increase the hard disk size of the virtual machine? - Ask Ubuntu](http://askubuntu.com/questions/88647/how-do-i-increase-the-hard-disk-size-of-the-virtual-machine)
1. 在命令行中执行 `VBoxManage modifyhd YOUR_HARD_DISK.vdi --resize SIZE_IN_MB`;
2. 在win系统的磁盘管理器中，格式化新增加的硬盘空间。


### 无法打开：VirtualBox：Could not find an open hard disk with UUID

在增加磁盘空间之前，幸好有做备份，所以在搜索到以下两个类似问题后，大概知道问题出在.vbox文件，所以对比修改前的文件，发现缺失的内容正是错误提示中的信息。恢复后正常。

- [类似问题1 by 小可同学 - 博客园](http://www.cnblogs.com/sammyke/archive/2012/02/16/2353708.html)
- [类似问题2 VirtualBox upgrade trashed my virtual machine - Stack Overflow](http://stackoverflow.com/questions/5209881/virtualbox-upgrade-trashed-my-virtual-machine)

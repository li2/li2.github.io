---
layout: post
title: Android APP - 从远程FTP服务器下载文件到本地
category: Android
tags: [android-service, android-broadcast, android-adb]
---

2014年4月24日~2014年4月25日
这是android app，没有activity，接收系统的broadcast后自启动，以service形式运行于后台，依据某种规则、定时从远程FTP服务器下载文件，并更新本地文件（GPS Ephemeris Data File, for Instant Fix Function）。

源码来自同事，编码风格优，程序逻辑优。
阅读源码、学习及调试：先建立一个临时的ftp，理清代码的整个执行流程；然后使用正确的ftp完成工作。
涉及知识：android Runnable, Thread, Broadcast, Service. TODO

<!-- more -->

------

## 程序设计流程

需要一个线程（**检查线程**）定时检查文件是否需要更新，判断依据是文件是否存在、文件是否过期、wifi是否连接；

- 若需要更新，则需要另外一个线程（独立于主线程的**下载线程**，这是启动下载线程的惟一情况），完成FTP登录、FTP下载、本机文件替换。下载结束后，不论成功还是失败后，则将检查列入计划表。
- 若不需要更新，则将检查列入计划表。

上述是将检查线程列入计划表（scheduleRunnable）的2种情况。

什么情况下启动检查线程（startRunnable）？

- wifi连接的broadcast;
- 屏幕亮的broadcast;
- 由系统调用的service onStartCommand()方法。（因为app没有activity，是以service的形式运行于后台）


什么情况下停止检查线程（stopRunnable）？当停止检查线程时，停止下载线程。

- wifi停止的broadcast;
- 屏幕灭的broadcast;
- 由系统调用的 onDestroy()方法。

## 搭建测试用FTP服务器

[set up ftp server on ubuntu 详细步骤](https://help.ubuntu.com/10.04/serverguide/ftp-server.html)

- 安装FTP Server `sudo apt-get install vsftpd`;
- 修改配置文件，使系统上已存在的用户具有FTP的访问权限：`/etc/vsftpd.conf  local_enable=YES  write_enable=YES`;
- 然后重启FTP Server：`sudo service vsftpd restart`.

### ftp login error log: 尝试连接“ECONNREFUSED - Connection refused by server”失败
server端口号是20（数据传输端口），client端口号是21（控制传输端口）。
一般保留client端端口号不设置，如果设置为20，将会导致该错误。

### app error log: FTPException [code=550, message= Failed to change directory.]
登录到ftp后，代码会执行 `changeDirectory(path)` 改变路径，该API对应FTP protocol中的命令`CWD path`
而向API传入的路径是`"/ftp"`, 正确的路径应该是`"ftp"`或者`"/home/weiyi/ftp"`.
路径的表示法要和unix一致。
对比 Runtime.getRuntime().exec(cmd). TODO 

### 通过浏览器登录FTP服务器
[How to connect to an FTP server from a Web Browser](http://www.speedguide.net/faq_in_q.php?qid=208)
`ftp://user:passwd@ftpserver_url/path`


## app编译、安装、启动

### 在anroid source code中编译apk
`android$ make -j8 EPODownload `, 定义于Android.mk中`LOCAL_PACKAGE_NAME := EPODownload`

### adb install error: INSTALL_PARSE_FAILED_INCONSISTENT_CERTIFICATES
adb install -r 无法解决，只有卸载再安装。
[错误解释1 by 念茜的CSDN博客](http://blog.csdn.net/yiyaaixuexi/article/details/6251245)
[错误解释2](http://stackoverflow.com/questions/3185444/how-to-deal-with-install-parse-failed-inconsistent-certificates-without-uninstal)

### adb install error: INSTALL_FAILED_DEXOPT
在system/app下面的apk是经过优化的，dex文件不会打包到apk中，dex文件会被优化后，生成odex文件。
找到未优化过的apk，即在out/target/product/generic/obj/APPS/下找到对应的APP：
android/out/target/product/sirfsocv7/obj/APPS/EPODownload_intermediates/package.apk.unaligned
[详细解释 by xiaoyaovsxin的CSDN博客](http://blog.csdn.net/xiaoyaovsxin/article/details/8216452)

### 通过adb命令启动app的service, activity
启动service：`adb shell am startservice -n package/.service`, 
package和service定义于AndroidManifest.xml:`package="com.mitac.EPOdownload"`, `<service android:name=".FTPService"  />`
所以具体的命令是：`adb shell am startservice -n com.mitac.EPOdownload/.FTPService`

启动activity：`adb shell am start -n package/activity`
activity定义于AndroidManifest.xml:`<activity android:name=".MainActivity">`

[更多的adb命令参考官网Android Debug Bridge](http://developer.android.com/tools/help/adb.html)
[或者参考eoe移动开发者社区](http://my.eoe.cn/876641/archive/21406.html)

### 不具有/mnt/sdcard写权限
`Caused by: java.io.FileNotFoundException: /mnt/sdcard/packedDifference.f2p7enc.ee: open failed: EACCES (Permission denied)`
删除AndroidManifest.xml中的`android:sharedUserId="android.uid.system"`，然后重新编译apk。
可以解决问题，但原因不明。TODO

---
layout: post
title: 为Ubuntu14.04部署Android App的Eclipse开发环境
category: Android
tags: [android-eclipse]
---

首先要说明的是，在ubuntu上部署开发环境费老劲了，几乎一天时间。**其实最费事的是有些东西只能用公司的代理下载，我自己的网，虽然配置了google-goagent，但时好时坏，呵呵**。而windows就没那么多磕磕绊绊。

现在google主推android studio，官网已经不提供eclipse+plugin打包下载了，但项目还在用eclipse，所以我也得把环境部署在自个的笔记本上，需要分别下载：

- 官网下载安装eclipse；
- 安装ADT Plugin，[官网教程](http://developer.android.com/sdk/installing/installing-adt.html)提供了2种方法，通过`Eclipse > Help > Install New Software...`，点击右上角的`Add...`按钮，会弹出来的`Add Repository`对话框，在`Location:`条目下输入URL链接；或者点击`Archive...`按钮选择压缩包；
- [官网下载Android SDK](http://developer.android.com/sdk/index.html)，只提供了基础开发工具，还需要通过Android SDK Manager下载其它工具；
- 配置eclipse的Android SDK路径；


## 部署Java Development Kit

在ubuntu环境下，需要安装`openjdk-7-jre`. 否则打开eclipse时遇到错误提示：

> A Java Runtime Environment(JRE) or Java Development Kit(JDK) must available in order to run Eclipse. No Java virtual machine was found after searching the following locations:


## 在64bit Ubuntu中安装32bit support library

在64bit OS上使用eclipse，及这个版本的`android-sdk_r24.1.2-linux`SDK时，会遇到错误：

> Error executing cannot run program .../build-tools/21.1.1/aapt": error=2, No such file or directory

[Android Install System Requirements](https://developer.android.com/sdk/index.html)提到

> Tested on Ubuntu® 14.04, Trusty Tahr (64-bit distribution capable of running 32-bit applications).

[Error: Cannot run aapt](http://stackoverflow.com/questions/18041769/error-cannot-run-aapt)  这个问答告诉我们通过`file`和`ldd`命令查看aapt的属性：

```sh
$ file build-tools/22.0.1/aapt
build-tools/22.0.1/aapt: ELF 32-bit LSB shared object, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.15, not stripped
 
$ ldd build-tools/22.0.1/aapt
    linux-gate.so.1 =>  (0xf7740000)
    librt.so.1 => /lib/i386-linux-gnu/librt.so.1 (0xf761a000)
    libdl.so.2 => /lib/i386-linux-gnu/libdl.so.2 (0xf7615000)
    libpthread.so.0 => /lib/i386-linux-gnu/libpthread.so.0 (0xf75f9000)
    libz.so.1 => /lib/i386-linux-gnu/libz.so.1 (0xf75e3000)
    libstdc++.so.6 => /usr/lib/i386-linux-gnu/libstdc++.so.6 (0xf74fe000)
    libm.so.6 => /lib/i386-linux-gnu/libm.so.6 (0xf74d2000)
    libgcc_s.so.1 => /lib/i386-linux-gnu/libgcc_s.so.1 (0xf74b4000)
    libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xf7309000)
    /lib/ld-linux.so.2 (0xf7741000)
```

所以我们**需要安装32bit support library for 64 bit System.**

[Android studio build error on ubuntu install](http://stackoverflow.com/questions/27078052/android-studio-build-error-on-ubuntu-install)   这个问答直接给出了需要安装的包：`apt-get install libc6:i386 libgcc1:i386 libstdc++6:i386 libz1:i386`

问题是，如何从ldd信息中得到需要安装的包？
[Is there a way to determine what packages or libraries should be loaded to support an executable?](http://unix.stackexchange.com/questions/101824/is-there-a-way-to-determine-what-packages-or-libraries-should-be-loaded-to-suppo)

> Run ldd as you have, then manually install those dependencies.
> For example `libpango-1.0.so.0 => /usr/lib/i386-linux-gnu/libpango-1.0.so.0 (0xb702f000)`
> For me would mean installing `apt-get install libpango:i386` and praying that the version in Debian unstable was good enough.


如果遇到权限错误，

```sh
chown -R YOUR_USERNAME:YOUR_USERNAMEchown -R YOUR_USERNAME:YOUR_USERNAME platform-tools/  tools/ platforms/  platform-tools/adb
```

## 部署在14.04上遇到的错误

上述问题针对12.04，在14.04上遇到的是另外的问题：ldd命令提示`not a dynamic executable`，那就不理会它，直接安装缺失的包，却又提示`Oracle JDK 7 Is NOT installed`，[解决方法](http://askubuntu.com/questions/414885/oracle-jdk-7-is-not-installed-error)：

```sh
sudo dpkg -P oracle-java7-installer
sudo apt-get -f install
```

## [运行Android Virtual Device](http://xmodulo.com/how-to-run-android-emulator-on-ubuntu-or-debian.html)


## Eclipse theme

UI Theme - `https://raw.github.com/guari/eclipse-ui-theme/master/com.github.eclipseuitheme.themes.updatesite`
Editor Theme - `http://eclipse-color-theme.github.io/update/`


## 代理设置

Android SDK Manager 如果更新失败则需设置代理：`Tools > Options...`，在弹出的对话框中输入server和port.

Eclipse 在install new software时一直pending也需要设置代理，特别需要注意的是，在设置了`HTTP`和`HTTPS`的host和port后，**需要把`SOCKS`条目的内容清掉，否则仍然连不上网**。参考：
[stackoverflow: Eclipse not connecting to internet via proxy](http://stackoverflow.com/questions/17338212/eclipse-kepler-not-connecting-to-internet-via-proxy)

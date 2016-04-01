---
layout: post
title: 使用Samba在Ubuntu和Windows间进行网络共享
category: 工具
tags: []
---

Samba是Ubuntu和Windows进行网络共享的工具，比如分享打印机，互相之间传输资料文件。

##在ubuntu上的配置步骤

------
作者：Ubuntu中文社区
原文网址：http://wiki.ubuntu.org.cn/Samba

> 1. 安装 （若已安装，略过）
sudo apt-get install samba

> 2. 为每个用户创建共享目录（个人更习惯在win上访问整个用户目录，所以略过此步骤）
mkdir /home/username/sharefolder
chmod 777 /home/username/sharefolder

> 3. 备份并编辑smb.conf允许网络用户访问代码：
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf_backup
sudo gedit /etc/samba/smb.conf
搜寻这一行代码：
; security = user
用下面这几行取代：
security = user
username map = /etc/samba/smbusers 
将下列几行新增到文件的最后面，假设允许访问的用户为username，共享文件夹的路径为/home/sharefolder
[username]
comment = ubuntu share folder for username
path = /home/username/sharefolder
valid users = username
read only = no  

> 4. 为用户建立samba密码
sudo smbpasswd -a username 

> 5. 启动
sudo /etc/init.d/smbd restart



## 在win上的配置步骤

------

- 右键单击“我的电脑”
- 在弹出的菜单中选择“映射网络驱动器”
- 在弹出的“映射网络驱动器”对话框中选择一个未使用的“驱动器”盘符，在“文件夹”中填入 \\IPAddress\username
- 在弹出的“正在连接到IPAddress”的对话框中输入smb.conf配置的valid users，及smbpasswd设置的密码；

或者通过热键Win+R调出“运行”


然后远程ubuntu上的共享文件夹就被映射为本地win上的网络驱动器。


## 不允许一个用户使用一个以上用户名和一个服务器或共享资源的多重连接

------
如果建立映射时遇到错误“不允许一个用户使用一个以上用户名和一个服务器或共享资源的多重连接”，需要断开之前的映射，重新建立。

产生这个错误的原因是 [网络文件夹目前是以其他用户名和密码进行映射的](http://blog.csdn.net/nxx_168/article/details/7522303)
被映射的网络共享文件夹所在的机器给不同的共享文件夹设置了不同的用户访问权限，而目前连接的机器与被映射的机器已经用另一个用户建立了连接，从而导致了此错误。
在cmd.exe中输入 net use 即可看到当前已建立的连接。

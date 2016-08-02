---
layout: post
title: 如何靠自己抢到新西兰WHV名额？
category: 碎笔
tags: []
---

## 环境准备

- 小区 20M 长城带宽
- VPS 虚拟主机；
- Mac Book + Terminal.app；
- Chrome 浏览器；
- SwitchyOmega 代理插件；
- AutoFill 自动填表插件;



## 时间节点

- 05.23 04:30 登陆网站
- 05.23 07:17 抢到表格
- 05.23 07:35 付款成功
- 05.25 09:49 收到确认信，签证状态 Visa Decision：Pending
- 06.04 23:13 备齐材料
- 06.08 18:44 寄出材料
- 06.28 12:30 收到签证邮寄通知信，签证状态 Visa Decision：APPROVED
- 06.29 15:55 收到签证
- 08.02 14:19 飞往奥克兰


## VPS

### 什么是 VPS，为什么 VPS 可以帮助抢名额？

VPS 全称 Virtual Private Server，虚拟专用服务器。当你搭建了一个 VPS，就意味着你在远程有一台 24 小时不间断运行的计算机，因为在远方（*张大锤：「有一段时间我们仰卧河底 用另一个角度看时间流淌 你把所有的远方称作河流」*），所以用户无法直接连接键盘鼠标和显示器来控制它，只能通过远程的方式访问，比如 SSH。（*如果你不会用 SSH，并且发现河流的尽头是新西兰移民网站，那么就游过去。*）

通过 SSH 访问到远程主机后，通过代理（proxy）服务，就可以让远程主机替你完成网络请求。对比使用 VPS 前、后的网络访问情况，假设远程主机架设在澳洲悉尼：

- 直接访问：大陆局域网 -> 移民局官网 （嗷 中国来的 刷太狠 我鸭梨山大 设限 弹出去）
- VPS 访问：大陆局域网 -> 悉尼国际网 -> 移民局官网 （嗷 隔壁邻居 不姓王 请进）

因为代理服务使得移民局官网认为访问来自澳洲而非中国，所以不会针对澳洲设限（假如针对中国 IP 的访问限制确实存在的话）。理论上是这样，实际上也是这样，在 5.23 抢票时，直接访问的小伙伴始终不能login，而VPS的我却可以保持登陆状态。

关于VPS，下面我只讲三点：如何搭建 VPS，如何通过 SSH 访问 VPS，如何使用代理。


### 如何搭建 VPS 

略读以下三篇博文，大概了解搭建亚马逊 VPS 的过程：

- [亚马逊一年免费VPS申请及SSH安全代理搭建](http://blog.sina.com.cn/s/blog_67de9c540102uxk3.html)
- [Amazon AWS亚马逊云服务免费一年VPS主机成功申请和使用方法](http://www.freehao123.com/amazon-aws/)
- [Amazon AWS —— 免费的午餐不好吃](http://bropaul.com/post/amazon-aws-in-practice)

然后直接去亚马逊官网注册 http://aws.amazon.com/cn/free/，按照向导一步一步往下走，选择免费的、基础的服务，具不细表。惟一需要强调的是，亚马逊把创建 VPS 称作 Create Instance，在创建一个 Instance 之前，先选择区域：

![region](/assets/img/whv/whv1.png)

注意创建时下面的一行小字提示，「Note：Your ... lanuch in ... Sydney」，既然选择 VPS，当然距离新西兰越近越好：

![region](/assets/img/whv/whv2.png)

### 如何使用 SSH 访问 VPS

然后需要建立和 VPS 的连接，方法有很多种，针对 MAC Book，使用系统自带的 Terminal.app （Finder Go Utilities）。亚马逊官网有对应的教程 [使用 SSH 连接到 Linux 实例](http://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)

当然，在如下页面，点击 Connect，也可以看到具体步骤：

![region](/assets/img/whv/whv3.png)

- 因为 terminal 命令行对文件的操作是有路径要求的，如果你不懂如何切换目录（或者不懂如何在命令行中指定路径），请把 .pem 文件移动到 Home 路径下；
- 修改文件权限 chmod ....；
- 使用 ssh 建立连接 ssh -D 8888 -i ......

第三步和官网给出的命令略有不同， -D 8888 指定了代理端口，稍后会讲到。成功后你会看到：

![region](/assets/img/whv/whv4.png)


### 如何通过代理访问目标网站

需要使用代理插件 SwitchyOmega，

- New profile... 按如下填写（8888 就是上述命令行配置的端口）

    ![region](/assets/img/whv/whv5.png)

- 然后新建规则，为目标网站选择刚刚新建的 profile：

    ![region](/assets/img/whv/whv6.png)

- 最后选择使用该 profile：

    ![region](/assets/img/whv/whv7.png)

- 结果我们就可以访问某歌了：
    
   ![region](/assets/img/whv/whv8.png)


## 如何正确使用自动填表插件预先填写所有信息而不必担心双P？

安装 AutoFill 插件：

![region](/assets/img/whv/whv9.png)

完全为了保险起见，在移民局官网注册一个新的账号，登陆官网，填写表格，全部填练（虚）手（假）信息，每填一个表格就点击 AutoFill 按钮弹出如下对话框，圈 2 有一个 New... 选项，点击新建表名，然后点击圈 3 存储表格。

![region](/assets/img/whv/whv10.png)

全部表格填完之后，进入 AutoFill Options 选项界面，左下角会看到所有表格：

![region](/assets/img/whv/whv11.png)

选择其中一个表格，修改为正确的信息：

![region](/assets/img/whv/whv12.png)

因为我们填的全是练（虚）手（假）信息，所以可以submit，可以走到填写付款信息的界面，可以把信用卡号保存到表单里（如果你心大，甚至可以把安全码填上的，然而我并没有填）。
一旦把所有表格修改为正确信息，就不要在登陆官网填表练手了，因为一旦打开对应的网页，表格就自动被填写了，你若submit 便不是晴天。



## 还需要积聚 23 点阳光

你以为有了 VPS，有了自动填表就够了么？So naive！Sometimes 需要阳光。关于如何获得阳光，我是有话说的，这需要在过去的九个月里见到一个人二十三次，每一次见面都会增加  1 点阳光，于是在23日那一天，集满的 23 点阳光会在关键时刻抢到表格。但其实不是抢到的，当时我好想有一张表格，于是这23点阳光在2016年5月23日的7点17分爆发出一个虫洞，我看到了远方，远方递给我一份表格，事情就是这样的。

Your are my sunshine, 23 是远方，你在远方，我想念你。


## 总结

- 提前搭好环境；
- 提前填好表格；
- 正式开抢时，不论心情如何像过山车一样变化，都要保持刷新（多打开几个窗口）；
- 剩下的交给运气。

运气好的话，就可靠自己抢到名额，而不必花费昂贵的中介费用。

![region](/assets/img/whv/whv13.png)

![region](/assets/img/whv/whv14.png)

![region](/assets/img/whv/whv15.png)

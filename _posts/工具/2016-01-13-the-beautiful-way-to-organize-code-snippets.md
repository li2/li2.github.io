---
layout: post
title: 保存代码段的利器 Gist & GistBox
category: 工具
tags: [github]
---

GitHub提供的Gist服务堪称是**保存代码段的利器**！可以把gist当做一个小型仓库，具有版本控制，markdown。从网上的搜索的结果来看，好像在10年就推出了这项服务，真是知道的太晚了！请允许我大呼**『工具改变生活！』『工具改变程序员的生活！』**

项目中会有很多情况下可以直接复用以前写好的代码，之前是怎么保存并查找这些代码片段呢？我基本上是凭着记忆翻之前的工程然后拷贝，或者保存代码文件到云笔记中。**效率肥肠低**。

目前Gist提供搜索服务，但使用仍然不便，比如没有标签，也就没法子便捷地管理。但是存在一款 web app，名叫 GistBox（英雄并不总是孤独的）。在了解gistbox以下几个特性之后，你肯定会爱上它（分分钟上手，肥肠懂你）！

- web app 跨平台，只要有浏览器就可以；
- 它提供**标签管理**功能；
- 点击一下『copy』就可以拷走代码段；
- 它的chrome浏览器插件（GistBox Clipper）提供了**3种便捷方式**，可以在当前的网页上弹出『新建一条gist』的编辑对话框，使你免于登陆web app：
    - **自动识别网页中的代码段**，会在代码段右上角显示gistbox logo，点击它；
    - 选中网页中的文本，右键弹出的菜单中，点击“save as gist”；
    - 直接点击位于chrome导航栏上的 gistbox 按钮；

下面的截图来自官网：

![gistbox](/assets/img/util/gistbox.png)


gistbox使管理、添加、使用gist（代码段）变得肥肠容易（前提是利用好搜索、定义好标签），长时间积累必定受益匪浅。
当然除了保存代码段之外，可以在gist保存你想保存的任何东西，比如技术总结、填过的坑。但是感觉保存代码段才能体现它的优势。

------

- 官方说明：[GitHub - About gists](About gists - User Documentation)

    > Gists are a great way to share your work. You can share single files, parts of files, or full applications. You can access gists at https://gist.github.com.
    > Every gist is a Git repository, which means that it can be forked, cloned, and manipulated in every way.

- [GixBox 官网](GistBox - The Beautiful Way to Organize Code Snippets)
- [Here's a quick tutorial on how to save code snippets with the extension](GistBox - The Beautiful Way to Organize Code Snippets)

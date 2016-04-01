---
layout: post
title: 使用OSX Terminal的小技巧
category: 工具
tags: [osx, emacs]
---

## 如何在命令行中打开 Emacs 图形窗口？

新建一个脚本命名为 `em`，放到 `PATH` 路径中，修改权限为 `755`：

```sh
#!/bin/sh
/Applications/Emacs.app/Contents/MacOS/Emacs "$@"
```
参考：[How to launch GUI Emacs from command line in OSX?](http://stackoverflow.com/questions/10171280/how-to-launch-gui-emacs-from-command-line-in-osx)
2016-01-26


## 如何列出两级文件夹目录树？

```sh
$ sudo brew install tree

$ cd sdk/samples/android-21
$ tree -L 2

.
├── legacy
│   ├── ApiDemos

```
参考：[How to list all the files in a tree (a directory and its subdirs)?](http://askubuntu.com/questions/15419/how-to-list-all-the-files-in-a-tree-a-directory-and-its-subdirs)




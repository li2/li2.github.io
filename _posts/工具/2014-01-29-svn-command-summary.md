---
layout: post
title: SVN命令总结
category: 工具
tags: []
---

- 从远程版本库中检出一份到本地，commit log 需要用到 username: 

        svn checkout /remote/url/ /local/path/ --username yourname

- 本地版本库的当前状态，在 commit 前必须确保被更改的文件是需要提交的文件：

        svn status

- 获取某个版本号的详细修改信息：    

        svn log --verbose -r 20

- 本地版本库两个版本之间的差异，仅仅显示文件列表：

        svn diff --summarize -r 19:20

- 导出本地版本库中两个版本之间的差异，包括完整的路径，一个开源的脚本文件：

        svn-rev-diff-export.sh -u <path> -f revFROM -t revTO -o <path>

- 恢复本地版本库被更改的文件，可用于 commit 时忽略被无意或误操作更改状态的文件：

        svn revert file

- 通知本地版本库新增的文件：

        svn add file
        svn add --depth=empty PATH

- 提交本地版本库的更改到远程：

        svn commit

- 本地版本库被“锁”而无法 commit, 需要“解锁”

        svn cleanup
        若解锁失败，尝试删掉根目录下 .svn/lock 文件

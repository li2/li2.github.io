---
layout: post
title: Git使用总结
category: 工具
tags: []
---

## git push 每次都要输入用户名和密码
原因是git clone URL使用的是HTTPS，需要**更改为SSH**，并**更新本地的git地址**。
[Git push requires username and password](http://stackoverflow.com/questions/6565357/git-push-requires-username-and-password)
>A common mistake is cloning using the default (HTTPS) instead of SSH. You can correct this by going to your repository, clicking the ssh button left to the URL field and updating the URL of your origin remote like this:
> `git remote set-url origin git@new.url.here`

完成上述两个步骤，再次push会提示一个错误：

```bash
Warning: Permanently added the RSA host key for IP address '' to the list of known hosts.
Permission denied (publickey).
fatal: Could not read from remote repository.
```
接下来需要**[按照官网的教程生成SSH key](https://help.github.com/articles/generating-ssh-keys/)**
这样就可以避免每次push需要输入用户名和密码。

<!-- more -->

*整理于2015-03-21*

------

## 多个SSH Key问题
但是一个SSH Key只能用于一个Github账号，如果试图把同一个key贴到另一个账号上，Github会提示错误。**我们需要为不同的账号生成不同的key**。

解决办法是建立配置文件`~/.ssh/config`

```sh
# GitHub Account 1
Host github-as-user1	// 这个名字在稍后用来测试连接：ssh -T git@github-as-user1
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_1

# GitHub Account 2
Host github-as-user2
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_2
```

特别需要注意的是，此时`~/.ssh/`路径下可能没有`id_rsa_1`文件，所以**在生成ssh key的命令中，需要加`-f`选项指明文件，会新建不存在的文件，如果你尝试手动建立这个文件的话，[会出现问题](http://stackoverflow.com/a/29315364/2722270)**：

```sh
ssh-keygen -t rsa -f ~/.ssh/id_rsa_work -C "weiyi.just2@gmail.com"
```

参考：
[Multiple GitHub Accounts & SSH Config](http://stackoverflow.com/a/17158985/2722270)
[git生成ssh key及本地解决多个ssh key的问题](http://riny.net/2014/git-ssh-key/)


## 多个key存在时，可能会push到错误的key账号

假如Github用户`user1`把本地代码添加到一个远程仓库`whatever.git`,并且命名为`origin`，

```sh
git remote add origin git@github.com:user1/whatever.git
```

参考我们上面配置的`~/.ssh/config`，可以把这行命令改写为：

```sh
git remote add gb-user1 git@github-as-user1:user1/whatever.git
```
想要提交`master`分支到`user1/whatever`仓库时，就可以执行`git push gh-user1 master`.

`gh-user1`是**[远程仓库在本地的简称](http://git-scm.com/book/zh/v1/Git-基础-远程仓库的使用#远程仓库的删除和重命名)**，可以改为你想要的名字，越简单明了越好，比如`li2.me`，假如出现
> fatal: remote gh-user1 already exists

可以`git remote rm gh-user1`

解决办法参考：
[How to work with multiple ssh keys](http://stackoverflow.com/a/8924826/2722270)

*整理于2015-03-28*

## 全局性地更换电子邮件地址

```sh
$ git filter-branch --commit-filter '
        if [ "$GIT_AUTHOR_EMAIL" = "old@email" ];
        then
                GIT_AUTHOR_NAME="new name";
                GIT_AUTHOR_EMAIL="new@email";
                git commit-tree "$@";
        else
                git commit-tree "$@";
        fi' HEAD

git update-ref -d refs/original/refs/heads/master
git push --force --tags origin 'refs/heads/*'
```
[全局性地更换电子邮件地址](http://git-scm.com/book/zh/v1/Git-工具-重写历史)
[Change the author of a commit in Git](http://stackoverflow.com/a/750182/2722270)

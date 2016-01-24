---
layout: post
title: Shell FAQs - 进程
filename: 2014-05-01-Shell-FAQ-Process.md
category: Unix
tags: [shell]
---

Shell进程相关问题汇总。

## Shell 进程

### crontab 重复执行任务

比如：每个工作日早8点定时自动重启Ubuntu电脑；

``` bash
#修改/etc/crontab$
#minute hour    mday    month   wday    user    command
0       8       *       *       1-5     root    shutdown -r now
```

<!-- more -->

或者把命令放在脚本中，这样修改时就无需动crontab文件：

``` bash
#minute hour    mday    month   wday    user    command
0       8       *       *       *       root    /usr/local/bin/crontab_reboot.sh

#/usr/local/bin/crontab_reboot.sh
#!/bin/sh

# get day(1~7) of week, $()命令置换
day=$(date +%u)

# if day less than 6, then it's a workday
if [ $day -lt 6 ] ; then
    shutdown -r now
fi
```

参考

- [How do I set up a Cron job? - Ask Ubuntu](http://askubuntu.com/questions/2368/how-do-i-set-up-a-cron-job)

  > Put a shell script in one of these folders: /etc/cron.daily, /etc/cron.hourly, /etc/cron.monthly or /etc/cron.weekly.
  > If these are not enough for you you can add more specific tasks eg. twice a month or every 5 minutes or... go to the terminal and type: `crontab -e`
  > This will open your personal crontab (cron configuration file),

- [How to restart every 30 minutes automatically? - Ask Ubuntu](http://askubuntu.com/questions/243546/how-to-restart-every-30-minutes-automatically)
- [shell script to check if today is a weekend](http://www.digitalinternals.com/190/20130330/shell-script-to-check-if-today-is-a-weekend/)
- 《shell脚本学习指南-13.6.4-crontab》
- [第十六章 例行性工作排程 (crontab) 鳥哥的Linux私房菜](http://linux.vbird.org/linux_basic/0430cron.php)



## 版本

li2 于上海闸北
2014-04-15 ~ 2014-05-15, v1

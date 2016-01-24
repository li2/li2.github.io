---
layout: post
title: Shell FAQs - 文本查找-替换-排序
filename: 2014-05-01-Shell-FAQ-Search-Substitute-Sort-Text.md
description: Shell 文本查找、替换、排序相关问题汇总。
category: Unix
tags: [shell]
---

## sed与awk优劣比较

[原帖由 jixunuli 于 2006-2-17 10:54 发表-Shell-ChinaUnix.net](http://bbs.chinaunix.net/thread-702042-1-1.html)

> 虽然两者都是处理文本的，但是 两者各有所长，有的功能用sed实现起来比较方便，另一些则用awk方便。
> 对，waker 版主举的就是一个例子。 他那个问题用 sed 很容易解决，但是用 awk 就比较费劲。
> 其实这也就是我学习 Perl 的原因，
> shell 下工具众多，功能也互相重复，
> 最头疼的是，这些重复部分的语法还各不相同，（比如 grep awk sed 都有正则表达式匹配的功能，但是三者的正则表达式语法就不相同）。
> 最最最头疼的是，每个工具还分 GNU 版和不是 GNU 版，之间的差别也很大，
> 最最最最最头疼的是，即使都是 GNU 版，那么版本号的细微差别也会带来很多差别。
> 但是，用 Perl 做这些事，统统都能办到，而且统统都不太复杂。

<!-- more -->

## 打印字符串的前几个字符

[Different ways to print first few characters of a string in Linux - The UNIX School](http://www.theunixschool.com/2012/05/different-ways-to-print-first-few.html)

``` bash
$ cat file
Linux
Unix
Solaris

while read line
do
    echo ${line:0:3}                   # ${line:0:3}是内建命令，从0编号，提取字符串line的前3个字符
done < file

$ cut -c 1-3 file                      # cut剪下指定的字符范围
$ grep  -o "^..." file                 # --only-matching, show only part of a line matching PATTERN
$ grep -o '^.\{3\}' file               # 区间表示式 .\{3\} 表示出现3次点号
$ awk '{ print substr($0,1,3) }' file  # awk的字符串提取函数substr，从1编号
$ sed 's/\(...\).*/\1/' file           # sed执行文本替换
$ sed 's/\(.\{3\}\).*/\1/' file
Lin
Uni
Sol
$
```

## 替换字符串 

[Using grep and sed to find and replace a string - Stack Overflow]( http://stackoverflow.com/questions/6178498/using-grep-and-sed-to-find-and-replace-a-string)

``` bash
grep -rl oldstr path | xargs sed -i 's/oldstr/newstr/g'     或者
find /path -type f -exec sed -i 's/oldstr/newstr/g' {}  \;  或者
find /path -type f | xargs sed -i 's/oldstr/newstr/g'
```

- grep 选项`-l, --files-with-matches`  print only names of FILEs containing matches，仅 每行一个 列出文件名。
如果没加`-l`，sed会提示错误`sed: can't read path/file No such file or directory`。
grep默认输出格式是：`/path/from/grep/: pattern in this line`。
选项`-Z, --null` print 0 byte after FILE name，打印匹配pattern的行。和grep没加任何选项时效果一致。
xargs把grep的输出一行一个传递给sed，sed从中读取文件名，直至遇到空白符，
所以sed读到的是`/path/from/grep/:` 文件名中多了一个冒号，所以sed报错。

- grep 选项 ` -i[SUFFIX], --in-place[=SUFFIX]` edit files in place (makes backup if extension supplied)
如果不加`i`，只在终端打印替换的效果，不会修改原文件。

- > `{} \;` The braces are the current filename as exec'd by find and without them sed will complain.
- `grep -rl oldstr path | xargs sed -i 's/oldstr/newstr/g' /dev/null` 会提示 `sed: couldn't edit /dev/null: not a regular file`.


## 查找包含2个、不在同一行的关键词的文件

使用egrep 1条命令就可以完成：

``` bash
grep -Erw "pattern1 | pattern2" .    或者
egrep -rw "pattern1 | pattern2" .
```

- 选项`-E, --extended-regexp` PATTERN is an extended regular expression (ERE) 使用扩展式grep。
- 选项`-w, --word-regexp` force PATTERN to match only whole words 强制匹配整个单词。
- 命令来源 [How to grep -w for 2 words that might or might not occur in the same line? -- Stackoverflow](http://stackoverflow.com/questions/17525777/how-to-grep-w-for-2-words-that-might-or-might-not-occur-in-the-same-line)


使用grep，需要2条命令：

``` bash
find . -type f -exec grep -rl pattern1 {} \; | xargs  grep -Z pattern2    或者
grep -rl pattern1 . | xargs grep pattern2 .
```
- 命令来源 [How to search files where two different words exist? - Unix - Stackexchange](http://unix.stackexchange.com/questions/67794/how-to-search-files-where-two-different-words-exist)


## 版本
li2 于上海闸北
2014-04-15 ~ 2014-05-15, v1

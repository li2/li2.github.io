---
layout: post
title: 「Shell脚本学习指南笔记」第07章-输入输出-文件与命令执行
filename: 2014-04-15-notes-Classic-Shell-Scripting-chapter-07.md
category: Unix
tags: [shell]
---

本章的主要内容：
如何处理输入/输出；
命令替换，如何使一个命令的输出作为参数；
引用；
命令执行顺序，内建shell命令；
 
<!-- more -->
 
## 使用read读取行

------

`read [-r] variable ...`
读取标准输入，通过shell内部的字段切割器`$IFS`进行切分，按顺序赋值给变量列表variable...
 
- 若分隔出的内容多于变量数，则将剩余的内容赋值给最后一个变量。
- 若输入行以反斜杠`\`结尾，认为是**续行符line continuation**，则继续读取下一行。
- 若使用选项`-r`，行尾的`\`仅作字面意义，赋值给变量。
- 若在管道中使用read，那么会在新的进程内执行该shell命令，任何read设置的变量，不能被父shell使用。
  管道中间内的循环也是这样。**父子shell  subshell**  TODO

使用简单的循环处理/etc/passwd：
 
```bash
while IFS=: read user pass uid gid fullname homedir shell
do
    ...     处理每行的用户记录
done < /etc/passwd
```
 
这个循环并不是说“当IFS等于冒号时，便读取……”，而是通过IFS的设置，让read使用冒号作为字段分隔符。仅改变read所继承环境的IFS值，不改变循环体里使用的IFS值。参考“表6-3：POSIX内置的shell变量”。

```bash
# 重定向的错误用法，每次循环时，shell会再次打开/etc/passwd，即每次都只读取文件的第一行。
while IFS=: read user pass uid gid fullname homedir shell < /etc/passwd
do
    ...     处理每行的用户记录
done

# 比较容易读取，但使用cat会损失一点效率。
cat /etc/passwd |
while IFS=: read user pass uid gid fullname homedir shell
do
    ...     处理每行的用户记录
done
```
 
通过管道把命令的输出传递给read，这个技巧当read用在循环中时，格外有用：
 
```bash
# 第3章 使用此脚本来复制整个目录树
find /home/testuser/ -type d -print         |  输出指定路径下的目录名称列表，一行一个打印
    sed 's;/home/testuser/;/home/backup/;'  |  修改目录名称，这里使用分号作为界定符
        sed 's/^/mkdir /'                   |  插入mkdir命令
            sh -x                              以shell跟踪模式执行
# 使用循环可以容易地完成，而且从程序员角度看更自然
find /home/testuser/ -type d -print         |  输出指定路径下的目录名称列表，一行一个打印
    sed 's;/home/testuser/;/home/backup/;'  |  修改目录名称，这里使用分号作为界定符
        while read newdir
        do
            mkdir $newdir 
        done
```


## 重定向

------

`set -C` 打开shell的禁止覆盖（noclobber）选项，防止文件被意外截断。
所以当`>`重定向到打开此功能的文件时失败。而`>|`可忽略此选项。
 
行内输入（inline input）`<<`, `<<-`
TODO
 
**文件描述符处理**
 
`make 1> results 2> ERRS` 把make命令的标准输出传递给results文件，标准错误传递给ERRS文件。
`make 1> results 2> /dev/null` 直接舍弃错误信息。
`make > results 2>&1` 重定向的默认文件描述符是标准输出，所以可以如此`> results`省略。此时文件描述符1已经是results文件，可以用`&1`引用。所以这条命令是将标准输出和错误都写入了results文件。**`2>&1`必须连在一起。**
 
**使用exec改变shell的文件描述符(I/O重定向)**
 
`exec 3< /temp/file`
`read variable <&3` 从/temp/file中读取
 
**使用exec在当前shell下执行指定程序**
TODO
 

## printf的完整介绍

------

` printf  "format-string"  [argumens ...] `
 
printf对转义序列的处理：
 
- 在`format-string`中按转义处理；
- 在`arguments...`中按字面意义打印；除非在格式化字串中使用`%b`，相应参数字串中的转义序列会被解释。
 
更多细节查阅P162.
 
 
## 波浪号展开与通配符

------

shell有两种与文件名相关的展开：
 
- 波浪号展开（tilde expansion）
- 通配符展开（wildcard expansion），又称，全局展开（globbing），又称，路径展开（pathname expansion）
 
波浪号展开的目的是改变路径：
`cd ~/` 进入当前用户的根目录，同`cd $HOME`; `cd ~weiyi/` 进入指定用户的根目录。
 
**通配符展开式**
 
| 通配符     | 匹配                     |
|------------|--------------------------|
| `?`        | 任意一个字符             |
| `*`        | 任意多个字符             |
| `abc`      | a, b, 或c                |
| `.,;`      | 句号、逗号、或分号       |
| `-_`       | 破折号, 或下划线         |
| `a-z`      | 任意一个小写字母         |
| `0-9`      | 任意一个数字             |
| `!0-9`     | 任意一个非数字字符       |
| `0-9!`     | 任意一个数字或叹号       |
 
`ls *` 会列出当前目录下的所有文件，不需要像MS-DOS中使用`*.*`;
执行通配符展开时，UNIX Shell会忽略文件名开头是点号的文件，这类是隐藏文件，通常用作系统配置文件或启动文件。
`echo .*` 显示隐藏文件
`ls -la` 列出所有文件，包括隐藏的，选项`-a`表示全部。
 
**点号开头的隐藏文件只是习惯用法，kernel不认为与其它文件有不同。**
 
 
## 命令置换（command substitution）

------

“命令置换”本身是可执行的shell命令，shell执行该命令后，用结果替换命令本身。命令置换 有两种形式： P170
 
 
- 使用反引号（或称重音符号`），把要执行的命令框住，但和双引号混合使用时，变得复杂不易阅读；
-  使用`$()`

    ```bash
    $ echo "outer + $(echo inner - $(echo "nested quote here")  - inner) + outer"
    outer + inner - nested quote here - inner + outer
    ```
 
**使用for循环比较不同目录下的相同文件的差异（比如版本比较）**
 
```bash
for i in $(cd /old/code/dir ; echo *.c)     产生/old/code/dir下的文件列表
do                                       
    diff -c /new/code/dir/$i $i             新版本与旧版本比较
done | more
```
 
**为head命令使用sed**
 
```bash
# head -- 打印前n行
# P68， 版本1：  head N file
count=$1
sed ${count}q "$2"

# P172，版本2：  head -N file
# 当以head -10 foo.xml调用这个脚本时，最终以sed 10q foo.xml被调用。
count=$(echo $1 | sed 's/^-//')
shift
sed ${count}q "$@"
```
 
**创建邮件列表**
**简易数学expr**
TODO
 
 
## 引用

------

引用（quoting）是用来防止shell将某些你想要的东西解释成不同的意义。P176
 
**反斜杠转义**
这是引用单一字符最简单的方式。
 
**单引号**
强制将单引号之间的所有字符看作字面上的意义。**当你希望shell完全不做任何处理时，使用单引号。**
 
```bash
$ echo 'here are some metacharacters: * ? [abc] $x \*'
here are some metacharacters: * ? [abc] $x \*
```
 
**双引号**
双引号，也把括起来的字符序列看作一个字符串，但不同于单引号的是，会确切地处理括起来的转义字符、变量、算术、命令置换。**当你希望将多个串视为一个串，同时需要shell做些处理的时候，使用双引号。**
 
```bash
$ name="I'm li2."
$ echo "\$name is \"$name\", '$(echo Hello World.)'"
$name is "I'm li2.", 'Hello World.'

oldvar="$oldvar $newvar"    将newvar的值附加到oldvar变量后。
```

 
## 执行顺序与eval

------

好像是在讲shell程序的解释器，就像c语言的解释器怎么去读取c语言一样。P177
**eval语句**
**subShell与代码块**
TODO
 
 
## 内建命令（built-in）

------

由shell本身执行命令，而不是在另外的进程中执行外部程序。P183
 
表7-9：POSIX的Shell内建命令
 
| 命令       | 说明                                             | 特殊 |
|------------|--------------------------------------------------|------|
| `:`        | 只做参数的展开                                   |   S  |
| `.`        | 读取文件，并于当前shell中执行文件的内容          |   S  |
| `cd`       | 改变工作目录                                     |      |
| `command`  | todo                                             |      |
| `continue` | 立即跳出本次循环，开始循环的下一次               |   S  |
| `break`    | 循环退出                                         |   S  |
| `return`   | 从函数中返回                                     |   S  |
| `exit`     | 退出shell                                        |   S  |
| `true`     | 逻辑值，表示成功                                 |      |
| `false`    | 逻辑值，表示失败                                 |      |
| `exec`     | 以给定的程序取代shell，或者改变shell的IO         |   S  |
| `export`   | 建立环境变量                                     |   S  |
| `getopts`  | 处理命令行的选项 todo                            |      |
| `set`      | 设置选项或者位置参数 todo                        |   S  |
| `shift`    | 移位命令行参数                                   |   S  |
| `jobs`     | 列出后台的工作                                   |      |
| `kill`     | 传送信号                                         |      |
| `trap`     | 设置信号捕捉程序                                 |   S  |
| `pwd`      | 打印工作目录                                     |      |
| `read`     | 从标准输入中读取一行                             |      |
| `readonly` | 设置变量为只读                                   |   S  |
| `umask`    | 设置、显示文件权限掩码                           |      |
| `unset`    | 删除别名定义                                     |   S  |
| `wait`     | 等待后台工作完成                                 |      |
| `times`    | 针对shell和子shell，显示用户与系统cpu时间的累计  |   S  |
 
 
第2章（p28）提到了：shell识别三种基本命令：内建命令（built-in commands）、shell函数（shell functions）、外部命令（external commands）。
内建命令由shell本身执行，处于同一进程中；内建命令部分是shell执行的基本要素，或者是提高shell执行效率。分为特殊special和一般regular。
外部命令，建立一个新的进程（即子shell subshell），在PATH列出的目录中寻找命令并执行，执行结束后从子shell中返回。
 
命令查找的次序：special-built-in > shell function > regular-built-in > external
这种查找顺序允许定义shell function以代替regular-built-in。
 
```bash
# 更改路径时，shell的提示号能包含当前路径的最后一部分。
# 因此定义一个cd函数，完成这件事情。
~/shellstudy$ cat cd
cd () {
    command cd "$@"    #command使shell可以访问系统cd命令，以避免调用时递归，导致Segmentation fault.
    x=$(pwd)
    printf "Built-in Commands: \$PWD=%s\n" $x
    PS1="${x##*/}\$ "
    printf "Built-in Commands: \$PS1=%s\n" $PS1
}
cd "$@"

# 执行结果
~/shellstudy$ ./cd $HOME
Built-in Commands: $PWD=/home/weiyi
Built-in Commands: $PS1=weiyi$

# 脚本执行后，在console中打印，对比上述：脚本也只是console的一个子进程
~/shellstudy$ echo $PWD
/home/weiyi/shellstudy

weiyi@MitacRD1:~/shellstudy$ echo $PS1
\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\u@\h:\w\$
```
 
当shell函数需要被多个脚本调用时，可以将它们放在单独的文件中，该文件只要能够读取即可。在其它脚本中以点号`.`来引用。
 
 
 
## 第8章 产生脚本

------

前7章是各种小例子示范，而第8章是一个复杂的程序。就好象给你讲完了四则运算，然后考你微积分。
TODO


## 版本
li2 于上海闸北 
2014-04-15 ~ 2014-05-15, v1

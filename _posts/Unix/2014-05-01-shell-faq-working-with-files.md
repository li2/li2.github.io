---
layout: post
title: Shell FAQs - 文件处理
filename: 2014-05-01-Shell-FAQ-Working-with-Files.md
description: Shell 文件处理相关问题汇总。
category: Unix
tags: [shell]
---
 
## 查找并拷贝文件，包括完整的路径（保持文件目录结构）
find and copy files with whole path (preserve folder structure)

``` bash 
#从当前目录中查找名为"c-files* "的文件，排除.o二进制文件，并拷贝完整路径到destination文件夹
find . -type f \( -iname "c-files*" ! -iname "*.o" \) -print0 | rsync -av --files-from=- --from0 ./  ./destination

#从当前目录中查找名为"c-files* "的c和h文件， 并拷贝完整路径到destination文件夹
find . -type f -name 'c-files*.[ch]'  -print0 | rsync  -av --files-from=- --from0 ./  ./destination
```

<!-- more -->

参考
 
- 上述命令来源[ commandlinefu.com/commands/view/1481/rsync-find ]( http://www.commandlinefu.com/commands/view/1481/rsync-find )
 
- [ Find command: Exclude / Ignore Files ( Ignore Hidden .dot Files ) ]( http://www.cyberciti.biz/faq/find-command-exclude-ignore-files/ )
    >  The parentheses must be escaped with a backslash, `\(` and `\)`, to prevent them from being interpreted as special shell characters.   反斜杠防止圆括号被shell解释为特殊字符，`\( \)`是子表达式（理解对吗？）
    > 选项 `-type f`仅查找文件。
    > 选项 ` -or`或，`-and`且，`-not`和`!`非。
 
-  find命令参考《shell脚本学习指南-10.4.3-find命令》
 
- 选项说明
    > ` -v, --verbose`               increase verbosity 显示拷贝的详细信息；
    > ` -a, --archive`               archive mode; equals -rlptgoD (no -H,-A,-X)（？）。
    > ` -0, --from0`                 all *-from/filter files are delimited by 0s
    > ` --files-from=FILE`           read list of source-file names from FILE
 
- [Copying Directories with "rsync"]( http://linux.about.com/b/2010/08/31/copying-directories-with-rsync.htm)
    >  One of the useful features of rsync is that when you use it copy directories, you can exclude files in a systematic way.
 
- [Rsync用法、实例、详解]( http://www.linuxany.com/archives/226.html)
- 问题： 如何表达文件名字的中间部分？
 
 
[使用exec完成相同的任务： Regex find and copy in bash (preserving folder structure)?- Stack Overflow](http://stackoverflow.com/questions/2839114/regex-find-and-copy-in-bash-preserving-folder-structure)：

``` bash
find ./ -type f \( -iname "c-files*" ! -iname "*.o" \) -exec cp --parents "{}"  ./destination \;
```
 
 
##  查找并拷贝文件，不包含路径

``` bash
find /path/to/search/ -type f -name "regular-expression-to-find-files" | xargs cp -t /target/path/
```
 
上述命令来源  [ how to move or copy files listed by find command in unix ]( http://stackoverflow.com/questions/17368872/how-to-move-or-copy-files-listed-by-find-command-in-unix )
仅拷贝文件，不包含路径。所以当有重名文件时，不被拷贝。 cp: will not overwrite just-created dest-file with  src-file
  
 
## 加上前缀/后缀以重命名文件
rename files with prefix/suffix
 
``` bash
rename 's/(.*)$/prefix.$1/'  old filename
```
 
参考
 
- 上述命令来源[bash - How to rename with prefix/suffix? - Stack Overflow]( http://stackoverflow.com/questions/208181/how-to-rename-with-prefix-suffix)
    >  If rename isn't available and you have to rename more than one file, shell scripting can really be short and simple for this. For example, to rename all .jpg to prefix*.jpg in the current directory:
          for filename in *.jpg; do mv "$filename" "prefix_$filename"; done;
 
- 问题： 上述命令`$1.suffix`修改的结果是name.type.suffix。 name.type 如何修改为 name-suffix.type?
  
 
## 查找并删除文件 find and delete files

``` bash
find . -type f -name "*.c" -print0 | xargs -0 rm
```
 
命令来源[Find and delete .txt files in bash](http://stackoverflow.com/questions/12604468/find-and-delete-txt-files-in-bash)
当文件名中含有空格时，最好使用 -print0 的方式，以生成ASCII NUL字符串结束符。
> `find . -type f -name "*.c" | xargs rm` That has the potential go gag on file names with spaces. If their find/xargs combo handles it, this is better: `find . -name "*.txt" -print0 | xargs -0 rm` to create ASCII NUL terminated strings.

``` bash
$ find . -type f -name "*.c"
./find   .c
$ find . -type f -name "*.c" | xargs rm                 #文件名含有空格时，rm失败. rm ./find\ \ \ .c可以删除。
rm: cannot remove `./find': No such file or directory
rm: cannot remove `.c': No such file or directory
$ find . -type f -name "*.c" -print0 | xargs -0 rm      #rm成功
$ find ./1 -type f -name "*.c" | wc -l
0
```
 
下面的命令也可以删除，但难记。

``` bash
$ find . -type f -name "*.c" -exec rm {} \;
```
 
## 新建文件
 
使用cat复制终端的输入（按ctrl+D结束输入end of the file）

``` bash
cat newfile
```

使用echo重定向

``` bash 
echo "Hello World, I'm li2." > newfile
```

## 版本
li2 于上海闸北 
2014-04-15 ~ 2014-05-15, v1

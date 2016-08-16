---
layout: post
title: Count Lines of Code
category: 工具
tags: [command-line]
---

Have you ever wondered how many lines of code have you written? Especially project that you expended a lot of your energy in it. Here is a command line tool which will help you count programmatically:

> cloc counts blank lines, comment lines, and physical lines of source code in many programming languages.

You can [get it on Github](https://github.com/AlDanial/cloc). The following is my project which I do all of the coding:

```sh
weiyiWorkCell:DVR weiyi$ cloc app/src/main/
     113 text files.
     113 unique files.                                          
       0 files ignored.

http://cloc.sourceforge.net v 1.64  T=0.60 s (188.2 files/s, 29565.2 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Java                            73           2516           1822          11289
XML                             40            332             26           1767
-------------------------------------------------------------------------------
SUM:                           113           2848           1848          13056
-------------------------------------------------------------------------------
```
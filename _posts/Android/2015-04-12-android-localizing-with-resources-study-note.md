---
layout: post
title: 「Android编程权威指南笔记」Android应用本地化
category: Android
tags: [android-resources]
---

本地化是一个基于设备语言设置，为应用寻找合适资源的过程：排除不兼容的目录；按优先级表筛选不兼容目录。
常用优先级：

> 移动国家代码MCC > 语言代码 > 布局方向

```bash
res/values/         # 保留默认资源，否则当android无法找到匹配资源，而默认资源又不存在时，应用将崩溃。
res/values-zh/      # 创建带有目标语言配置修饰符的资源子目录。
res/values-land/    # 横屏模式。
res/values-zh-land/ # 多重配置修饰符，修饰符必须按照优先级顺序排列。
```

几个原则：

> - drawable目录通常按照屏幕显示密度要求，具有三类修饰符：-mdpi、-hdpi和-xhdpi。不过，Android决定使用哪一类drawable资源并不是简单地匹配设备的屏幕显示密度，也不是在没有匹配的资源时直接使用默认资源。
- 无需在res/drawable/目录下放置默认的drawable资源。
- 资源的名字只能由小写字母组成并且不能包含空格。
- 无论是在XML还是在代码中引用资源，引用都不应包括文件的扩展名；因此在同一子目录下，不能以文件的扩展名为依据，来区分命名相同的资源文件。
- 所有资源都必须保存在res/目录的子目录下。尝试在res/目录的根目录下保存资源将会导致编译错误。
- res子目录的名字直接与Android编译过程绑定，因此无法随意进行更改。
- 无法在res/目录下创建多级子目录，当资源文件很多时，唯一能做的就是有意识的对资源命名，使其可按文件名进行排序，以便于查找某个特定文件。布局文件通常以其定义的视图类型名作为前缀，如activity_、dialog_，以及list_item_等。


---

整理笔记时参考了如下资料：

- 《Android编程权威指南》第11章应用本地化， Bill Phillips  Brian Hardy著，王明发 译。人民邮电第1版。
    英文版书名《Android Programming - The Big Nerd Ranch Guide》
- 官方文档 [Localizing with Resources](http://developer.android.com/guide/topics/resources/localization.html)

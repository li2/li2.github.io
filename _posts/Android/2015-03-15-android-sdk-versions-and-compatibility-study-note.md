---
layout: post
title: 「Android编程权威指南笔记」SDK版本与兼容 
category: Android
tags: []
---

## 第6章 Android SDK版本与兼容

Android开发者必须花费时间保证向后兼容。尽管Android以及第三方库提供了相应的兼容性编程支持。常常需要学习完成同一件事的两种方法，以及如何将这两种方法进行整合。而有时虽然只学习一种方法，但学习起来却异常地复杂，因为我们要努力实现至少两套开发需求。

只需学会查阅SDK文档，不断学习新的东西并掌握它们即可。

```Java
@TargetApi(11)	// 禁止Android Lint提示兼容性问题
@Override
protected void onCreate(Bundle savedInstanceState) {
    ......
    // 确保actionbar在Honeycomb或更高版本的设备上运行应用才会被调用
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        ActionBar actionBar = getActionBar();
        actionBar.setSubtitle("title");
    }
}
```

<!-- more -->

---

整理笔记时参考了如下资料：

- 《Android编程权威指南》Bill Phillips  Brian Hardy著，王明发 译。人民邮电第1版。
    英文版书名《Android Programming - The Big Nerd Ranch Guide》

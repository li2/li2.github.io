---
layout: post
title: 「Android编程权威指南笔记」Android布局和组件
category: Android
tags: [android-layout]
---

## 第8章 使用布局与组件创建用户界面

### 样式、主题及主题属性

样式（style）是XML资源文件，含有用来描述组件行为和外观的属性定义。
主题（theme）是各种样式的集合。
使用主题属性引用（theme attribute reference），相当于告知Android运行资源管理器：“在应用主题里找到名为listSeparatorTextViewStyle的属性。该属性指向其他样式资源，请将其资源的值放在这里”。
`<TextView style="?android:listSeparatorTextViewStyle" />`
使得屏幕上的TextView组件看起来是以列表样式分隔开的。

### dp、sp以及屏幕像素密度

常见的属性：

- 文字大小（text size），指设备上显示的文字像素高度；
- 边距（margin），指定视图组件间的距离；
- 内边距（padding），指定视图外边框与其内容间的距离。

Android使用drawable-ldpi、drawable-mdpi以及drawable-hdpi三个目录下的图像文件自动适配不同像素密度的屏幕。
Android提供了密度无关的尺寸单位（density-independent dimension units），以使边距在不同屏幕密度的设备上获得同样大小的尺寸。

- dp：density-independent pixel，密度无关像素。在设置边距、内边距或任
何不打算按像素值指定尺寸的情况下，通常都使用dp这种单位。1dp单位在设备屏幕上总是等于1/160英寸。
- sp：scale-independent pixel，缩放无关像素。设置屏幕上的字体大小。

<!-- more -->

### Android开发设计原则

用16dp单位值设定边距尺寸。该单位值的设定遵循了Android的“48dp调和”设计原则。

### 布局参数

名称不以layout_开头的属性作用于组件。组件生成时，会调用某个方法按照属性及属性值进行自我配置。比如padding告诉组件：在绘制组件自身时，要比所含内容大多少。
名称以layout_开头的属性则作用于组件的父组件。我们将这些属性统称为布局参数。它们会告知父布局如何在内部安排自己的子元素。比如边距属性决定了组件间的距离。

### layout_weight属性的工作原理

对于水平布局，其子组件width设置为0dp，宽度按weight等比分配。习惯weight值加起来等于1.0或100。

---

整理笔记时参考了如下资料：

- 《Android编程权威指南》Bill Phillips  Brian Hardy著，王明发 译。人民邮电第1版。
    英文版书名《Android Programming - The Big Nerd Ranch Guide》

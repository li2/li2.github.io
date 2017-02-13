---
layout: post
title: Android RecyclerView
description: 
category: Android
tags: [android-view]
---


文章链接：[http://li2.me/2017/02/android-recycler-view.html](http://li2.me/2017/02/2017-02-11-android-recycler-view.html)

RecyclerView坑：item 间距越滑动越大。

RecyclerView 坑：`onCreateViewHolder` `parent.getHeight()` 0，动态添加的 Fragment 中的 RecyclerView。

ImageView - have height match width?

使用 `android.support.v7.widget.DividerItemDecoration` 添加分割线。

水纹效果 `android:background="?android:attr/selectableItemBackground"` 高亮效果

java.lang.IllegalStateException: Fragment already added

Android FragmentTransaction commit already called

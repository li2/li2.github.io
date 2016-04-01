---
layout: post
title: 如何学习 Android Animation？
category: Android
tags: [android-animation]
---

在 [Android 开发者网站](http://developer.android.com/)  搜索『animation』，通过『blog』过滤搜索结果，可以获得很多博文，我只摘录了前两页，并把它们分成了两类：


## 动画可以做成什么样子（第1类）


下述几篇博文通过如下3个方面向开发者展示『动画』：

- 给出一些指导原则，阐述为什么要如此做动画，或者这样的动画有什么好处；
- 给出 GIF 动图，直观地展示动画是什么；
- 给出实现这种动画的代码片段（关键类、方法、资源文件）或者实现思路；

**这非常棒，可以帮助我们很快建立动画的印象，了解某个名词代表的动画是什么样子，应该怎么去实现。而且代码片段很多是从开源项目中摘录的，意味着我们可以调试这些动画。**

### [2014-08-05 Material design in the 2014 Google I/O app](http://android-developers.blogspot.com/2014/08/material-design-in-2014-google-io-app.html)

这篇文章的作者是 Google I/O app 的主设计师，而 I/O app 的作用之一是**提供 Android 设计和开发的最佳实践**（it serves as a reference demo for Android design and development best practices）。文中特别提到了**最喜爱的 app 细节之一就和动画有关**：浮动操作按钮（floating action button）的状态根据“用户是否会出席当前页面的这个会议”而改变，状态改变时伴随着动画。文中详细地讲到了动画实现的步骤，虽未罗列代码，但可以从 [github-google-io-app](https://github.com/google/iosched) 获取源码，了解实现细节。

![io](/assets/img/android/learn-android-animation-io.gif)


### [2014-10-24 Implementing Material Design in Your Android app](http://android-developers.blogspot.com/2014/10/implementing-material-design-in-your.html)

这篇文章一半篇幅在讲**『Motion』：当用户触摸屏幕时，所有的变化应从接触点开始向外辐射，建立画面过渡时的关联和连续性**，这些必须是有意义且友好的。Materials 通过这种方式向用户提供反馈，使用户注意到变化，从而帮助到用户。（to focus attention, establish spatial relationships and maintain continuity. Materials respond to touch to confirm your interaction and all changes radiate outward from your touch point. All motion is meaningful and intimate, aiding the user’s comprehension.）。分别介绍了：

- **Activity & Fragment Transitions**：在两个页面间平滑过渡，常见的场景是从列表视图切换到详情视图；
- **Ripples**：一种波纹效果，波纹从触点开始向外扩散直至填充整个view，点击继承自 Theme.Material 的 button 就可以看到这种效果；
- **StateListAnimator**：View 状态改变时的动画；
- **Circular Reveal**：一个圆形扇面从触点开始向外辐射直至填充整个view，这种过渡效果常用于展示新的内容；
- **Interpolators**：插值器定义了动画改变的速率，比如允许 alpha, scale, translate, rotate 加速、减速、重复等。比如插值器 fast_out_slow_in， 加速开始、逐渐降速直至结束，这样的动画效果使得对象在整个运动轨迹中，在接近终点的位置耗费较多时间。而根据不同的场景，选择不同的插值器或者自定义插值器，就可以使动画更具意义，摆脱千篇一律的印象。

### [2015-05-28 Announcing the Material Design Showcase and Awards](http://android-developers.blogspot.com/2015/05/announcing-material-design-showcase-and.html)

这篇文章先回顾了14年6月首次公布 material design 时的愿景**『a single design system that can work across platforms and brands』**。但这些愿景和 material design 的设计思想全是通过『假想中的 App』 来演示的。然后转眼到了15年5月，也就是这篇文章发布的日期，世界已经发生了变化，很多 App 接受并通过 Android 5.0 SDK 和 AppCompat 实现了material design。

因此特别收集了 18 个符合 material design 的 app 向众人展示，其中 6 个 app 由于在 material design 某一方面做的特别出色而被授予了第一届 Material Design Awards. 因动画效果而获奖的是 Tumblr: Delightful Animation. 
**动画效果可以做成什么样子**，官方告诉你去看看 Tumblr.

### [2015-06-16 More Material Design with Topeka for Android](http://android-developers.blogspot.com/2015/06/more-material-design-with-topeka-for_16.html)

这篇文章是开源 Android App 项目 [Topeka for Android](https://github.com/googlesamples/android-topeka) 的说明文档，用来**演示 material design 的设计原则**，帮助开发者在不同的平台上建立统一的用户体验。(原文：It demonstrates that the same branding and material design principles can be used to create a consistent experience across platforms.)

Topeka 是一个趣味问答应用，包含9个种类的问题，以网格布局呈现，点击某个格子进入回答问题的页面，一个问题回答完后就切换到下一个问题，有选择题填空题等各种类型的问题（对应各种UI组件）。包含了很多动画元素：

1. Transitions: 很棒的 Activities 转场动画（great transitions between Activities）；

    ![transitioins](/assets/img/android/learn-android-animation-transitions.gif)

1.  Animations: 答题时有精心编排的动画，一旦答题就弹出一个浮动操作按钮，点击按钮提交答案就进入了下一个问题，这时会根据回答正确与否播放对应的动画；

    ![complex animations](/assets/img/android/learn-android-animation-complex.gif)

1. Property Animations：为 circular reveal 增加颜色渐变的动画（从FAB的颜色变成透明）以营造出消融的效果（Adding a color animation from the FAB's color to transparent creates a dissolve like effect to the circular reveal）。（上上一个gif包含这种效果，但太快了看不出效果，下面这个gif延长了时间，为了突出动画）；

    ![property animations](/assets/img/android/learn-android-animation-property.gif)


## 概述动画相关的类和接口（第2类）

- [2011-05-30 Introducing ViewPropertyAnimator](http://android-developers.blogspot.com/2011/05/introducing-viewpropertyanimator.html)
- [2011-02-24 Animation in Honeycomb](http://android-developers.blogspot.com/2011/02/animation-in-honeycomb.html)
- [2011-11-01 Android 4.0 Graphics and Animations](http://android-developers.blogspot.com/2011/11/android-40-graphics-and-animations.html)

前两篇文章先讨论了一个问题，已经有了能实现 move, scale, rotate, and fade 这些视图动画的 `android.view.animation` ，**为什么还要在 3.0 引入新 APIs？带来了哪些新特性？**然后进一步展示了这些新特性的强大便利之处。第三篇文章讨论了 **4.0 在 3.0 核心特性基础上增加的一小点儿改善**。（These articles discuss the new APIs added in 3.0 that make animation in Android easier, more powerful, and more flexible. The Android 4.0 improvements discussed below are small additions to these core facilities.）

这几文章排在搜索结果的前几位，可见其重要程度，作者是 Chet Haase，一个致力于图形和动画研究的 Android 开发者，可以从他的 [个人博客graphics-geek.blogspot.com](http://graphics-geek.blogspot.com)  阅读更多相关主题的博文。

需要说明的是 HONEYCOMB 3.0（ [about versions android-3.0 highlights](http://developer.android.com/about/versions/android-3.0-highlights.html) ） 发布于 2011.02，引入了New animation framework；3.1 发布于 2011.05；ICE_CREAM_SANDWICH 4.0 发布于 2011.11。**而这几篇博文的发布时间与相应的系统版本发布时间一致**，而且其中两篇文章在 [Android API Guides: Animation and Graphics](http://developer.android.com/guide/topics/graphics/index.html) 页面有推荐，所以真的应该**多关注和学习官方开发者的文章**。

[2015-04-21 Android Support Library 22.1](http://android-developers.blogspot.com/2015/04/android-support-library-221.html)
这篇文章介绍了 Android support library 22.1 版本带来的新特性。与动画有关的内容是：Lollipop android.R.interpolator 新增的几个 Interpolators 已经在 Support V4  得到支持。

**通读完上述几篇文章后，我们就能够了解到『动画可以做成什么样子』、『实现这些动画的类和接口，及技术的演变历史』。是不是有种高屋建瓴、运筹帷幄的感觉？我的感觉是，不至于淹没于 Android 文档的海洋中，被巨多的技术细节打的晕头转向。而且我觉得学习 demo 时在精不在多，所以应该先从官方的 sample 以及上述官方文章中的 demo 入手，同时查阅 API guides：**


## Android Training & Guides

Android 开发者网站提供的 Training：

- [Animating Views Using Scenes and Transitions](http://developer.android.com/training/transitions/index.html)
- [Adding Animations](http://developer.android.com/training/animation/index.html)
- [Defining Custom Animations](http://developer.android.com/training/material/animations.html)

Android 开发者网站提供的 API Guides：

- [Animation and Graphics Overview](http://developer.android.com/guide/topics/graphics/index.html)

有很多途径获取官方 sample：[Welcome to code samples for Android developers](http://developer.android.com/samples/index.html)，比如直接在 Android Studio 中导入：

![import sample](/assets/img/android/learn-android-animation-sample.png)


## 学习路线图

非官方的博文也有很棒的，比如下面这一篇：

> [Exploring Meaningful Motion on Android](https://medium.com/ribot-labs/exploring-meaningful-motion-on-android-1cd95a4bc61d) ，这是译文 [探索安卓中有意义的动画](http://segmentfault.com/a/1190000004182537) ，这是文章对应的开源项目 [hitherejoe/animate](https://github.com/hitherejoe/animate) ，1280颗星。

类似的一个开源项目 [lgvalle/Material-Animations](https://github.com/lgvalle/Material-Animations)，3840颗星。

但如果需要系统地学习某块知识点，从第一手和权威的角度讲，官网及其博客是开发者的不二选择。不要介意在阅读英文文档上花费的时间，这些都是值得的。看不懂的再查阅相关文章相佐证。

所以我花了大概三天时间，整理出这篇文章，制定了我的学习路线图：

1. 通读官方 blog（上述列出的）；
2. 学习官方 training（上述列出的）；
3. 学习 GitHub 高星（感兴趣的）项目（好多人推荐了好多非常好的项目）；
4. 阅读其它高质量的 blog 查缺补漏（善用 google）；

Go! Go! Go!


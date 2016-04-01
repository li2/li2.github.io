---
layout: post
title: 「译」Android ViewPropertyAnimator 介绍
category: Android
tags: [android-animation]
---

> 原文 [2011-05-30 Introducing ViewPropertyAnimator](http://android-developers.blogspot.com/2011/05/introducing-viewpropertyanimator.html) 发表于 2011.05（和 [3.1](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#HONEYCOMB_MR1) 发布的时间同步），作者是 Chet Haase，一个致力于图形和动画研究的 Android 开发者，可以从他的 [个人博客graphics-geek.blogspot.com](http://graphics-geek.blogspot.com)  阅读更多相关主题的博文。
>
> 译者说：
> 这篇文章介绍了在 3.1 中增加的和动画机制有关的类 `ViewPropertyAnimator`。先从三个方面做了论述，虽然 3.0 时已经有了能实现任意对象和属性动画的 ObjectAnimator，但仍然有必要提供一个新类，以提供一种更加简明易读的方式来实现 View 多个属性的并发动画。然后介绍了这个新类 ViewPropertyAnimator 的用法。并且提供了一个例子和视频来展示实际的效果。
>
> 本文非全文翻译，亦非逐字逐句翻译。
>
> 相关文章：
> 
> - [如何学习 Android Animation？](http://li2.me/2016/01/how-to-learn-android-animation.html)
> - [「译」Android Animation in Honeycomb by Chet Haase（Android 3.0系统中的动画机制）](http://li2.me/2016/01/android-animation-in-honeycomb.html)
>
> weiyi.li [li2.me](li2.me) 2016-02-02 ~ 2016-02-03

------

早先一篇文章「Animation in Honeycomb」（[原文链接](http://android-developers.blogspot.com/2011/02/animation-in-honeycomb.html)、[译文链接](http://li2.me/2016/01/android-animation-in-honeycomb.html)）介绍了 3.0 增加的 APIs `android.animation`，它比以前更容易实现对象动画和属性动画，包括 `View.class` 在3.0中新增的几个属性。为了使 View 的这几个新增属性做动画更加容易，3.1 又新增了一个工具类。

如果你不知道 View 的新增属性，需要先回顾上一篇文章中标题为「View properties」的一节内容，然后阅读本文。


## 回顾：使用 ObjectAnimator

使用 ObjectAnimator 为 View 的属性做动画，需要创建一个 Animator，设置动画的时间参数、值参数、目标对象、目标属性的名字，就可以播放动画了。比如，渐隐某个视图 `myView`，可以对属性 `alpha` 施加动画：

```java
    ObjectAnimator.ofFloat(myView, "alpha", 0f).start();
```

这虽然容易，但仍有很大的改进空间。考虑到视图属性会经常用来做动画，应该引入一些新的 API，使得实现动画的代码**简单**且**可读**，同时还需改善视图属性动画的性能。3.0 动画机制中关于视图属性的性能有三处值得改善：

其一，我们在一个没有内建「属性概念」的语言中做属性动画。由于系统运行时对「属性」没概念，因此 ObjectAnimator 需要通过特殊手段把**「代表属性名称的字符串」**变成对**「目标对象属性的 setter 方法」**的调用。比如遇到字符串 `"alpha"` 就知道该去调用 `View.setAlpha()`。这种间接手段被称为反射 reflection 或者 JNI（java native interface），虽然可靠但会带来额外开销。基于对视图属性的了解，我们应该提供一些 API 使得动画可以直接操作属性，避开这种间接手段。

*译注：如果不理解上述内容，应该回顾上篇文章「Animation in Honeycomb」中 ObjectAnimator 一节对「隐含的条件」的解释。*

其二，有些情景中可能需要**同时**对一个 View 的**多个属性**做动画。虽然所有动画共享计时机制而不会因计时产生额外开销，但多个属性动画需要构造多个 Animator 对象。既然我们知道要对 View 的哪几个属性做动画，就应该考虑将多个动画合并成一个，以减少开销。一种方法是使用 `PropertyValuesHolder`，它允许仅构造一个 Animator 对象，就可以实现多个属性动画。但这种方法需要更多的代码，对简单的动画而言过于笨重。应该有一种新的方法既能实现我们的期望，同时代码也是简单易读的。

*译注：如果不理解上述内容，应该回顾上篇文章「Animation in Honeycomb」中 AnimatorSet 一节内容。*

其三，View 的每一个属性都会执行一些操作以确保视图对象及其父对象在合适的时候重绘。比如， 在 x 轴平移视图，会使视图过去和现在的位置失效，以确保父视图正确地重绘它。而在 y 轴平移视图，也会做类似的操作。如果这两个属性同时改变，还做双份的重绘工作就显得多余。所以应该将这些工作合并在一起。`ViewPropertyAnimator` 可以做到。



## 介绍 ViewPropertyAnimator

ViewPropertyAnimator 提供了一种并行属性动画的简单方法。只需构建一个 Animator（*译注：针对上述2*）；能够直接修改目标视图的属性值（*译注：针对上述1*）；优化重绘工作，使多个属性的重绘仅发生一次（*译注：针对上述3*）。

那么上述渐隐视图的代码可以这样写：

```java
    myView.animate().alpha(0);
```
简单、易读。且容易组合多个属性动画，比如移动到 (50, 100):

```java
    myView.animate().x(50).y(100);
```

关于上述两行代码有几件事情值得一说：

- `animate()`：从调用 View 对象的 animate() 方法开始，该方法返回一个 ViewPropertyAnimator 实例，然后就可以调用它的方法来设置动画参数。
- 自动开始：要注意上述两行代码并未调用 start() 方法，因为在新 API 中动画启动是隐式的，一旦声明完，这些动画就「立刻」「一起」启动了。如果要深究细节的话，其实也并非「立刻」启动，而是在下一次 UI 刷新时。ViewPropertyAnimator 正是利用了这个时间差，把所有声明的动画收集在一起。只要还在继续声明新的动画，那么所有动画就会继续等待下一次 UI 刷新。一旦完成声明，所有动画就在下一次 UI 刷新时一起启动了。
- 流式接口：ViewPropertyAnimator 拥有流式接口（[Fluent interface](https://en.wikipedia.org/wiki/Fluent_interface)），允许函数链式调用，使得在一行代码中可以构建多个属性动画。

代码确实简单且易读了，但性能提升从何而来呢？



## 性能考量

性能提升一，能够直接修改目标视图的属性，不再赘述。

性能提升二，只需构建一个 Animator 就可以实现多个动画。
如果采用 AnimatorSet 实现的话，需要这样做，这段代码创建了两个 Animator 对象（也就是两个独立的动画），然后通过 AnimatorSet 同时播放：

```java
    ObjectAnimator animX = ObjectAnimator.ofFloat(myView, "x", 50f);
    ObjectAnimator animY = ObjectAnimator.ofFloat(myView, "y", 100f);
    AnimatorSet animSetXY = new AnimatorSet();
    animSetXY.playTogether(animX, animY);
    animSetXY.start();
```
或者使用 `PropertyValuesHolder`，只构造一个 Animator 对象来同时实现多个属性动画：

```java
    PropertyValuesHolder pvhX = PropertyValuesHolder.ofFloat("x", 50f);
    PropertyValuesHolder pvhY = PropertyValuesHolder.ofFloat("y", 100f);
    ObjectAnimator.ofPropertyValuesHolder(myView, pvhX, pvyY).start();
```

而使用 ViewPropertyAnimator 实现的代码已经在上节内容中提到了。

性能提升三，ViewPropertyAnimator 会在每一个动画帧，计算变化的属性值并设置给属性，然后只调用一次 invalidate 进行重绘。代替每一个属性单独计算、单独重绘。



## 一个例子

[点这儿下载](https://sites.google.com/site/androidcontentfromchet/downloads/VPADemo.zip) 或者 [点这儿查阅](https://github.com/li2/Learning_Android_Open_Source/blob/master/ApiDemos/app/src/main/java/com/example/android/apis/animation/ViewPropertyAnimator.java)。



## 其它

以上讲这么多，并不意味着 ViewPropertyAnimator 可以取代 3.0 中 animation 相关的 API。事实上 3.0 相关 API 为 ViewPropertyAnimator 甚至整个系统的动画功能提供了重要的支持。ObjectAnimator 可以灵活方便为任何对象和属性做动画。但当需要同时为 View 的多个属性（SDK提供的，非自定义扩展的）做动画时，ViewPropertyAnimator 会更方便。

还要注意的是：使用 ObjectAnimator 时并不需要太过担心性能，使用反射和 JNI 等带来的开销相对整个程序来讲都是较小的。使用 ViewPropertyAnimator 最大的优势也不在于性能的提升，对我来讲，而是它提供的简明易读的代码书写方式。

---
layout: post
title: 「译」Android Animation in Honeycomb（蜂巢3.0系统中的动画机制）
category: Android
tags: [android-animation]
---

> 背景： HONEYCOMB 3.0 （[about versions android-3.0 highlights](http://developer.android.com/about/versions/android-3.0-highlights.html)） 发布于 2011.02，引入了新的动画机制 android.animation；3.1 发布于 2011.05.
>
> 在 [Android 开发者网站](http://developer.android.com/)  搜索「animation」，通过「blog」过滤搜索结果，其中一篇文章 [2011-02-24 Animation in Honeycomb](http://android-developers.blogspot.com/2011/02/animation-in-honeycomb.html) 排在搜索结果第2位，可见其重要程度，作者是 Chet Haase，一个致力于图形和动画研究的 Android 开发者，可以从他的 [个人博客graphics-geek.blogspot.com](http://graphics-geek.blogspot.com)  阅读更多相关主题的博文。
>
> 这篇文章讨论了一个问题，已经有了能实现 move, scale, rotate, and fade 这些视图动画的 `android.view.animation` ，**为什么还要在 3.0 引入新 APIs `android.animation`？ 新 APIs 带来了哪些新特性？** 然后进一步展示了这些新特性的强大便利之处。
>
> 虽然是11年2月发布的，而现在已经是2016年的1月底了，虽然步子慢了五年，但3.x的动画依然没有过时，这篇文章风采依旧，非常值得初学者（我）学习。
> 本文并非全文翻译，亦非逐字逐句翻译，作为我自己的学习笔记，可谓是「总结式翻译」，梳理、备忘。**强烈建议直接阅读原文**。

------


Honeycomb 引入的新特性之一是全新的动画系统 `android.animation`，它比以前更容易实现对象动画（objects animation）和属性动画（properties animation）。


## Honeycomb之前的动画 ##

Honeycomb 之前，动画由 android.view.animation 实现，比如

- 视图的移动 move、缩放 scale、旋转 rotate、渐变 fade；
- 通过 `AnimationSet` 组合安排多个动画；
- 把动画指定给 `LayoutAnimationController`，当容器排列子视图时，会自动错开所有子视图动画的开始时间；（文末有注释）
- 使用 `Interpolator`（插值器，用来定义动画改变的速率），比如 AccelerateInterpolator 和 BounceInterpolator，使动画不再匀速改变，从而显得更自然。

如上述，honeycomb 之前可以实现**视图动画**，但也仅止于此，因为 honeycomb 之前动画只能操作视图对象（View Objects），缺乏一些关键功能的支持，对诸如 Drawable 的位置、背景色等这些视图属性无能为力。

之前的动画仅仅改变目标视图看起来的样子，而不会改变视图对象的属性。比如通过动画 TranslateAnimation 和 setFillAfter(true) 改变按钮的位置，动画仅仅在新的位置重绘按钮，而无法改变按钮在父视图中的位置。也就是说在新位置点击按钮无效。

基于这些（还有其它没有提到的）原因，Honeycomb 提供了全新的动画机制，新机制建立于**属性动画（property animation）**的概念之上。


## Honeycomb中的属性动画 ##

新动画机制不仅仅针对视图对象，也不仅仅针对对象的某些属性，也不局限于视觉效果。事实上，它所有的一切都是关于一段时间后**值的变化**，并把这些变化的值设置给**任何**目标对象和对象属性。因此，你可以：

- 移动或者渐变视图；
- 移动视图内的 Drawable；
- 动态地改变 Drawable 的背景色；
- 动态地改变任何数据类型的值，只需要告诉新机制：动画时长、自定义类型的动画过程值的计算方法、动画的起止值。有了这些，新机制就可以计算动画过程值并设置给目标对象（或属性）。

所以，通过新机制移动按钮是真的移动了按钮的位置。

接下来我会简单地介绍新机制中的几个关键类，适当时给出示例代码。想了解新机制工作的更多细节，就去研读 SDK sample ApiDemos.(译注：参考我这篇文章 [从 Android Sample ApiDemos 中学习 android.animation API 的用法](http://li2.me/2016/01/android-sdk-sample-api-demos-for-animation.html))


## Animator ##

[`Animator`](http://developer.android.com/reference/android/animation/Animator.html) 是新动画机制中的超类。子类 `ValueAnimator` 是核心计时引擎，子类 `AnimatorSet` 用来把多个动画编排成一个动画。一般不会直接使用 `Animator`，但它的一些属性和方法为子类所共有，诸如持续时间duration、开始延迟startDelay、监听器listener.

当你想在动画结束时执行一些操作，监听器就显得很重要了。为了监听动画的生命周期事件，需要实现接口 `AnimatorListener` 并注册给 animator。比如，

```java
    anim.addListener(new Animator.AnimatorListener() {
        public void onAnimationStart(Animator animation) {}
        public void onAnimationEnd(Animator animation) {
          // do something when the animation is done
        }
        public void onAnimationCancel(Animator animation) {}
        public void onAnimationRepeat(Animator animation) {}
    });
```
当然，考虑到你可能只需要监听某一个事件，意味着接口的其它方法不需要覆写，却占着空间可能会让「代码简洁控」抓狂，所以新机制提供了适配器类 AnimatorListenerAdapter，让你只需覆写关心的方法：

```java
    anim.addListener(new AnimatorListenerAdapter() {
        public void onAnimationEnd(Animator animation) {
          // do something when the animation is done
        }
    });
```

## ValueAnimator ##

`ValueAnimator` 是整个新机制的「主力」。它运行的内部计时循环（timing loop 文末有注释），使程序内的所有动画，在每次计时脉冲（timing pulse 文末有注释）发生时，根据当前的时间计算动画过程值，并设置给目标对象和属性。

它拥有的一些核心特性可以帮助它做到这一点：

- 知道每一个动画的计时详情（诸如起/止时间、持续时间、当前执行了多少时间）；
- 知道动画是否重复；
- 当前动画过程值算出来后，回调注册给它的监听器（`AnimatorUpdateListener`）；
- 拥有计算不同类型值的能力（`TypeEvaluator`）。

属性动画包含两个步骤：计算动画过程值，然后把这些值设置给目标对象或者属性。`ObjectAnimator`（稍后讲） 直接继承自 ValueAnimator，由它实现属性动画较为容易。但以下几种情况使用 ValueAnimator 反而更方便：

- 当对象没有提供某个属性的 setter 方法时；
- 当你仅有一个动画，并想把动画过程值设置给多个属性时；
- 或者仅仅是想使用一个简单的计时机制；

怎么使用 ValueAnimator 呢？比如，在 500ms 内从 0 变到 1：

```java
    ValueAnimator anim = ValueAnimator.ofFloat(0f, 1f);
    anim.setDuration(500);
    anim.start();
```

如前述，**新的动画机制不局限于视觉效果，而所有的一切都是关于一段时间后值的变化**。那么通过什么方式才能知道动画是否发生了呢？——答案是监听器：实现一个监听器 AnimatorUpdateListener 并注册给 ValueAnimator 实例，就可以在每一个动画帧（animation frame 文末有注释）被回调到，从而获取当前的动画值：

```java
    anim.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
        public void onAnimationUpdate(ValueAnimator animation) {
            Float value = (Float) animation.getAnimatedValue();
            // do something with value...
        }
    });
```

除了浮点数，还可以给其它类型的值添加动画，比如整数：

```java
    ValueAnimator anim = ValueAnimator.ofInt(0, 100);
```

XML 文件也可以实现相同的动画：

```xml
    <animator xmlns:android="http://schemas.android.com/apk/res/android"
        android:valueFrom="0"
        android:valueTo="100"
        android:valueType="intType"/>
```

新机制知道如何计算整数和浮点数的动画过程值，这是因为其内置了相关的计算方法。而对于其它类型的数据，比如 Point, Rect, 甚至自定义数据，新机制不知如何计算，但通过 TypeEvaluator（稍后解释），倒手就把计算动画过程值的责任交给了你：

```java
    Point p0 = new Point(0, 0);
    Point p1 = new Point(100, 200);
    ValueAnimator anim = ValueAnimator.ofObject(pointEvaluator, p0, p1);
```
除了动画时长外，还可以设置的动画参数有：

- setStartDelay(long)：播放延迟的时间；
- setRepeatCount(int)：播放重复的次数，0（默认值）不重复，正整数或者 Animation.INFINITE；
- setRepeatMode(int)：播放重复的模式，Animation.REVERSE 从相反方向重复，Animation.RESTART 从开始位置重复；
- setInterpolator(TimeInterpolator)：设置插值器，用来定义动画改变的速率。`TimeInterpolator` 是 `Interpolator` 的父接口，因此这里也可以使用 Interpolator 相关的实现。


## ObjectAnimator ##

ObjectAnimator 继承自 ValueAnimator。除了可以设置动画的时间参数和值参数外（这些 ValueAnimator 能做到的），它还可以设置目标对象和属性。比如，如果想渐隐某个对象 `myObject`，可以对属性 `alpha` 施加动画：

```java
    ObjectAnimator.ofFloat(myObject, "alpha", 0f).start();
```
这个例子中，虽然只设置了动画的终点值，但起始值默认是属性的当前值。

XML 文件也可以定义相同的动画：

```xml
    <objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
        android:valueTo="0"
        android:propertyName="alpha"/>
```

但不能在 xml 中设置目标对象，只能在加载动画资源后，由代码设置：

```java
    ObjectAnimator anim = AnimatorInflator.loadAnimator(context, resID);
    anim.setTarget(myObject);
    anim.start();
```

在使用 ObjectAnimator 之前，必须要理解一个隐含的假定，关乎属性及其 set/get 方法。这个假定是说：
当对某个对象的某个属性施加动画时，动画机制假定这个对象**具有和该属性名相关的 public `set` 方法**，并且属性具有合适的数据类型。还有，在构造 ObjectAnimator 时如果只设置了终点值，比如上述例子，动画机制必须要知道属性的当前值，因此，该对象也应该**具有相应的 public `get` 方法**以返回合适类型的数据。

因此，上述关于 alpha 的示例代码若想执行成功，对象 myObject 必须要有两个 public 方法：

```java
    public void setAlpha(float value);
    public float getAlpha();
```
简言之，某个对象若想为它的某个属性使用 ObjectAnimator，该对象必须满足动画机制「没有明说」的一个条件：提供该属性的 public set/get 方法。

特别说明：set/get 方法在 animation 运行时调用，因此它们和动画之间是非常弱的关系。而如果你的程序中又没有任何地方显示地调用这些 set/get 方法，而你恰好又使用了 ProGuard（代码混淆工具）或者其它代码优化工具（code stripping 代码剥离），那么这些工具便不会知道 set/get 在运行时被调用，很有可能就被剥离掉了。所以当你使用这些工具时，你有义务确保这些 set/get 方法的存在。


## View properties ##




## AnimatorSet ##


## TypeEvaluator ##


## BUT WAIT, THERE'S MORE! ##


------

## 译注：程序说明，英文单词词组 ##

### 关于 animation system
动画机制，文中大多简称「新机制」，是指 3.0 引入的 android.animation. 感觉此处译作「动画系统」不太合适，因为简称做系统时，很容易和 Android 系统混淆。


### 关于 timing loop
计时循环。

`android.animation.ValueAnimator` 定义了一个静态内部类 [`AnimationHandler`](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/animation/ValueAnimator.java#ValueAnimator.AnimationHandler)，它实现了 Runnable，在内部维护一个计时循环（和 `Choreographer` 有关 TODO），从而产生计时脉冲（timing pulse），动画之所以能「动」，就是计时脉冲在起作用，可认为是动画的「心脏」，功能类似于单片机的晶振。

程序内所有有效的动画共享同一个脉冲，以保证所有动画共享相同的时间，为了同步。每次脉冲发生时，每一个动画都会根据当前的时间以及拥有的插值器类型，计算出当前的动画过程值：`anim.doAnimationFrame(frameTime)`，最终会调用到 `mUpdateListeners.get(i).onAnimationUpdate(this)`。所以若注册监听器，就可以获得每次动画帧的动画值。


### 关于 animation frame
动画帧（这里肯定不能把 frame 当做框架来理解）。

上述讲到 timing loop 时提到了计时脉冲，在每次脉冲下，动画都会根据当前时间和插值器计算动画值，有了新值，就意味着动画发生了变化，所以，可以理解为一帧一帧的动画。


### 关于 animated values
动画过程值，动画中间值。

由于属性动画的本质是「值的变化」，所以当指定一个动画的起、止值时，在每一个动画帧，动画就会根据相应的计算方法算出一个值，这些值就是「animated values」。


### 关于 interpolator
插值器，按照某种算法，把一个值映射为另一个值。

public interface TimeInterpolator {}
public interface Interpolator extends TimeInterpolator {}
abstract public class BaseInterpolator implements Interpolator {}
直接继承自 BaseInterpolator 的子类有：
AccelerateDecelerateInterpolator, AccelerateInterpolator, AnticipateInterpolator, AnticipateOvershootInterpolator, BounceInterpolator, CycleInterpolator, DecelerateInterpolator, LinearInterpolator, OvershootInterpolator, PathInterpolator

搜索自 CNKI 词典：
interpolation
①插值法,内插法,内推法,插入法②插入,插入物③窜改
～by central difference 中差插值法
～by continued fractions 用连分式的插值法
～by convergents 收敛插值法
～by propotionaI parts 比例插值法
～error 内插误差
～error-filter 内插误差滤波器
～factor 内插因子
～formula 插值公式
～functions 插值函数
～line 内插行
～method 内插法,插入法
～models 插值模型
～of contours 等高线内插
～of gravity 重力内插
～oscillator 内插振荡器
～polynomial 插值多项式
～property 插入性质
～series 插值级数
～set 插入集[合]
～table 内插表
～technique 插入法,间插法
～theory 插值理论
- 来源：英汉科技大词库·第二卷 E-M


### 关于 staggered animation
automatically staggered animation start times：自动错开动画的开始时间。

staggered adj. 错列的；吃惊的。 [`LayoutAnimationController`](http://developer.android.com/reference/android/view/animation/LayoutAnimationController.html) 用于实现容器布局动画，容器内的子视图动画相同，但开始时间不同。所以这里的 staggered 应是「错开的」意思。


### 其它一些单词和词组

usher in 领进，引进。


## 一个哲学问题 ##


------

by
weiyi.li li2.me weiyi.just2@gmail.com
2016-01-28
禁止转载

**不把阅读英文文档当做翻译事业的程序员不是好厨子**
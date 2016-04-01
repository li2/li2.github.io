---
layout: post
title: 「译」Android Animation in Honeycomb by Chet Haase（Android 3.0系统中的动画机制）
category: Android
tags: [android-animation]
---

> 原文 [2011-02-24 Animation in Honeycomb](http://android-developers.blogspot.com/2011/02/animation-in-honeycomb.html) 发表于 2011.02（和3.0发布的时间同步），作者是 Chet Haase，一个致力于图形和动画研究的 Android 开发者，可以从他的 [个人博客graphics-geek.blogspot.com](http://graphics-geek.blogspot.com)  阅读更多相关主题的博文。
>
> 译者说：
> 这篇文章讨论了一个问题，已经有了能实现 move, scale, rotate, and fade 这些视图动画的 `android.view.animation`，**为什么还要在 3.0 引入新 APIs `android.animation`？ 新 APIs 带来了哪些新特性？** 然后进一步展示了这些新特性的强大便利之处。
>
> 虽然是11年2月发布的，而现在已经是2016年的1月底了，虽然步子慢了五年，但3.x的动画依然没有过时，这篇文章风采依旧，非常值得初学者（我）学习。
>
> 本文非全文翻译，亦非逐字逐句翻译，作为我自己的学习笔记，可谓是「总结式翻译」，梳理、备忘。
> **强烈建议阅读原文，文档和代码要结合阅读，以相互佐证和理解，同时运行 ApiDemos 中相关的示例，观察动画效果，修改参数再观察动画效果。**
>
> 相关文章：
> 
> - [如何学习 Android Animation？](http://li2.me/2016/01/how-to-learn-android-animation.html)
> - [从 Android Sample ApiDemos 中学习 android.animation API 的用法](http://li2.me/2016/01/android-sdk-sample-api-demos-for-animation.html)
> - [Android Animation Interpolator - Android 动画插值器分析](http://li2.me/2016/01/android-animation-interpolator.html)
>
> weiyi.li [li2.me](li2.me) 2016-01-28 ~ 2016-02-01


------


Honeycomb 引入的新特性之一是全新的动画系统 `android.animation`，它比以前更容易实现对象动画（objects animation）和属性动画（properties animation）。


## Honeycomb之前的动画 ##

Honeycomb 之前，动画由 android.view.animation 实现，比如

- 视图的移动 move、缩放 scale、旋转 rotate、渐变 fade；
- 通过 `AnimationSet` 组合安排多个动画；
- 把动画指定给 `LayoutAnimationController`，当容器排列子视图时，会自动错开所有子视图动画的开始时间（**文末有注释**）；
- 使用 `Interpolator`（插值器，用来定义动画改变的速率，**文末有注释**），比如 AccelerateInterpolator 和 BounceInterpolator，使动画不再匀速改变，从而显得更自然。

如上述，honeycomb 之前可以实现**视图动画**，但也仅止于此，因为 honeycomb 之前动画只能操作视图对象（View Objects），缺乏一些关键功能的支持，对诸如 Drawable 的位置、背景色等视图属性无能为力。

之前的动画仅仅改变目标视图的视觉效果（看起来的样子），而不会改变视图对象的属性。比如通过动画 TranslateAnimation 和 setFillAfter(true) 改变按钮的位置，动画仅仅在新的位置重绘按钮，而无法改变按钮在父视图中的位置。也就是说在新位置点击按钮无效。

基于上述（还有其它没有提到的）原因，Honeycomb 提供了全新的动画机制，新机制建立于**属性动画（property animation）**的概念之上。


## Honeycomb中的属性动画 ##

新动画机制不仅仅针对视图对象，也不仅仅针对对象的某些属性，也不局限于视觉效果。事实上，它所有的一切都是关于一段时间后**值的变化**，并把这些变化的值设置给**任何**目标对象和属性。

因此你可以完成很多事情，诸如：移动或者渐变视图；移动视图内的 Drawable；动态地改变 Drawable 的背景色；甚至动态地改变任何数据类型的值，只需要告诉新机制：动画时长、自定义类型的动画过程值的计算方法、动画的起止值，新机制就可以计算动画过程值并设置给目标对象和属性。

所以，通过新机制移动按钮是真的移动了按钮的位置。

接下来我会简单地介绍新机制中的几个关键类，适当时给出示例代码。想了解新机制工作的更多细节，就去研读 SDK sample ApiDemos.(译注：参考我这篇文章 [从 Android Sample ApiDemos 中学习 android.animation API 的用法](http://li2.me/2016/01/android-sdk-sample-api-demos-for-animation.html))


## Animator ##

[Animator](http://developer.android.com/reference/android/animation/Animator.html) 是新动画机制中的超类。子类 `ValueAnimator` 是核心计时引擎，子类 `AnimatorSet` 用来把多个动画编排成一个动画。一般不会直接使用 Animator，但它的一些属性和方法为子类所共有，诸如持续时间duration、开始延迟startDelay、监听器listener.

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

`ValueAnimator` 是整个新机制的「主力」。它运行的内部计时循环（timing loop **文末有注释**），使程序内的所有动画，在每次计时脉冲（timing pulse **文末有注释**）发生时，根据当前的时间计算动画过程值，并设置给目标对象和属性。

它拥有的一些核心特性可以帮助它做到这一点：

- 知道每一个动画的计时详情（诸如起/止时间、持续时间、当前执行了多少时间）；
- 知道动画是否重复；
- 当前动画过程值算出来后，回调注册给它的监听器（`AnimatorUpdateListener`）；
- 拥有计算不同类型值的能力（`TypeEvaluator`）。

属性动画包含两个步骤：计算动画过程值，然后把这些值设置给目标对象和属性。`ObjectAnimator`（稍后讲） 直接继承自 ValueAnimator，由它实现属性动画较为容易。但以下几种情况使用 ValueAnimator 反而更方便：

- 当对象没有提供某个属性的 setter 方法时；（译注：有疑惑不要紧，稍后讲到 ObjectAnimator 时就知道原因了。）
- 当你仅有一个动画，并想把动画过程值设置给多个属性时；
- 或者仅仅是想使用一个简单的计时机制；

怎么使用 ValueAnimator 呢？比如，在 500ms 内从 0 变到 1：

```java
    ValueAnimator anim = ValueAnimator.ofFloat(0f, 1f);
    anim.setDuration(500);
    anim.start();
```

如上述，**新的动画机制不局限于视觉效果，而所有的一切都是关于一段时间后值的变化**。那么通过什么方式才能知道动画是否发生了呢？——答案是监听器：实现一个监听器 AnimatorUpdateListener 并注册给 ValueAnimator 实例，就可以在每一个动画帧（animation frame **文末有注释**）被回调到，从而获取当前的动画值：

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

新机制知道如何计算整数和浮点数的动画过程值，这是因为其内置了相关的计算方法。而对于其它类型的数据，比如 Point, Rect, 甚至自定义数据，新机制不知如何计算，但通过 TypeEvaluator（稍后讲），转手就把计算动画过程值的责任交给了你：

```java
    Point p0 = new Point(0, 0);
    Point p1 = new Point(100, 200);
    ValueAnimator anim = ValueAnimator.ofObject(pointEvaluator, p0, p1);
```
除了动画时长外，还可以设置的动画参数有：

- setStartDelay(long)：播放延迟的时间；
- setRepeatCount(int)：播放重复的次数，0（默认值）不重复，正整数或者 Animation.INFINITE；
- setRepeatMode(int)：播放重复的模式，Animation.REVERSE 从结束位置重复，Animation.RESTART 从开始位置重复；
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

敏锐的读者可能已经发现了新动画机制的「瑕疵」：围绕属性做文章的新机制，如何处理那些连一个 public set/get 属性方法都没有的视图对象呢？
好问题！继续读下去。

类 [View](http://developer.android.com/reference/android/view/View.html)（查阅文档中 Animation 相关的说明） 在 Honeycomb 中得到了升级，增添了很多新属性，可以通过 set/get 方法访问这些属性，使得新动画机制可以应用于视图对象：

- translationX 和 translationY
- rotation, rotationX, 和 rotationY
- scaleX 和 scaleY
- pivotX 和 pivotY
- x 和 y
- alpha


## AnimatorSet ##

`AnimatorSet` 用来把多个动画编排成一个动画（作用类似于3.0以前的 AnimationSet）。比如你想编排一个这样的动画：先渐隐一个视图，结束后从侧边滑入另一个视图，滑入的同时渐显出来。为了实现这样的编排，首先需要拆分成几个独立的动画，接下来就有好几种选择了：(1) 在正确的时间手动播放对应的动画，或者给每个动画设置合适的播放延迟，总之你需要显示地、主动地设置合适的时间；(2) 如果觉得时间不好掌控，或者嫌麻烦，就使用 AnimatorSet，它提供的 APIs 使这些变得很简单：

- 同时播放：playTogether(Animator...);
- 顺序播放：playSequentially(Animator...);
- 通过 `AnimatorSet.Builder` 设置动画间的相对关系：with(), before(), after();
- play(Animator) 会创建一个 Builder;

因此上述动画可以这样实现：

```java
    ObjectAnimator fadeOut = ObjectAnimator.ofFloat(v1, "alpha", 0f);
    ObjectAnimator mover = ObjectAnimator.ofFloat(v2, "translationX", -500f, 0f);
    ObjectAnimator fadeIn = ObjectAnimator.ofFloat(v2, "alpha", 0f, 1f);
    AnimatorSet animSet = new AnimatorSet().play(mover).with(fadeIn).after(fadeOut);;
    animSet.start();
```
也可以在 XML 文件中实现上述动画。


## TypeEvaluator ##

上述 ValueAnimator 一节有说过一段话：「新机制知道如何计算整数和浮点数的动画过程值，这是因为其内置了相关的计算方法。而对于其它类型的数据，比如 Point, Rect, 甚至自定义数据，新机制不知如何计算，但通过 TypeEvaluator，转手就把计算动画过程值的责任交给了你。」

而 TypeEvaluator 是只定义了一个方法的接口：

```java
public interface TypeEvaluator<T> {
    public T evaluate(float fraction, T startValue, T endValue);
}
```
内置的处理浮点数的 FloatEvaluator 继承了该接口，而它的方法仅仅是 `y = kx + b` 的实现，非常简单：

```java
    public class FloatEvaluator implements TypeEvaluator {
        public Object evaluate(float fraction, Object startValue, Object endValue) {
            float startFloat = ((Number) startValue).floatValue();
            return startFloat + fraction * (((Number) endValue).floatValue() - startFloat);
        }
    }
```

那么这个方法是如何被调用到的呢？

```java
// ValueAnimator.class

    void animateValue(float fraction) {
        // 篡改播放进度 turns the elapsed fraction into an interpolated fraction.
        fraction = mInterpolator.getInterpolation(fraction);
	
        for (int i = 0; i < numValues; ++i) {
            // 根据篡改后的进度计算动画过程值 turn the interpolated fraction into an animated value.
            // 继续追代码，会追到 PropertyValuesHolder 和 Keyframes 两个类，深究不下去了 orz TODO
            mValues[i].calculateValue(fraction);
        }
    }
```

如果不是浮点数和整数，或者内置的处理整数的 IntEvaluator 和处理浮点数的 FloatEvaluator 不能满足你的要求，则必须显示地设置 TypeEvaluator，可以调用 `setEvaluator(TypeEvaluator)` 或者通过构造器 `ValueAnimator.ofObject(TypeEvaluator, Object...)`，比如计算 Point 的动画过程值：

```java
    public class PointEvaluator implements TypeEvaluator {
        public Object evaluate(float fraction, Object startValue, Object endValue) {
            Point startPoint = (Point) startValue;
            Point endPoint = (Point) endValue;
            return new Point(startPoint.x + fraction * (endPoint.x - startPoint.x),
                startPoint.y + fraction * (endPoint.y - startPoint.y));
        }
    }
```

现在使用它把在 ValueAnimator 这节内容中提到的例子补充完整：

```java
    Point p0 = new Point(0, 0);
    Point p1 = new Point(100, 200);
    ValueAnimator anim = ValueAnimator.ofObject(new PointEvaluator(), p0, p1);
```

译注：这篇文章发布于 2011.02，针对于 API level 11 的 3.0 系统，此后 L18 加入了 RectEvaluator， L21 加入了 PointFEvaluator、IntArrayEvaluator、FloatArrayEvaluator。
**所以上述几次提到的「不知道如何计算 Rect 的动画过程值」也并不是错误的，要特别注意技术文章的时效，查阅最新的文档和代码，以相互佐证。**
查阅 [TypeEvaluator Known Indirect Subclasses](http://developer.android.com/reference/android/animation/TypeEvaluator.html)。



## BUT WAIT, THERE'S MORE! ##

还有非常多的新特性可以说，但是限于文章篇幅和时间，这里就不再继续下去了。现在你应该和 ApiDemos「耍一耍」，潜心研究代码。

- 动画重复的一些特性；
- 动画生命周期事件的监听器；
- 在两个值以上做动画；
- 使用 Keyframe 定制更复杂的时值序列（time/value）；
- 使用 PropertyValuesHolder 指定多个属性并行动画；
- 使用 LayoutTransition 定制简单的布局动画；
- ......


------

## 译注：从代码角度理解一些特定的英文词组 ##

### 关于 animation system
动画机制，文中大多简称「新机制」，是指 3.0 引入的 android.animation。感觉此处译作「动画系统」不太合适，因为简称做系统时，很容易和 Android 系统混淆。
所以「新机制」在本文的语境下特指 android.animation.


### 关于 timing loop
计时循环。

`android.animation.ValueAnimator` 定义了一个静态内部类 [AnimationHandler](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/animation/ValueAnimator.java#ValueAnimator.AnimationHandler)，它实现了 Runnable 以维护计时循环，从而产生计时脉冲（timing pulse），动画之所以能「动」，就是计时脉冲在起作用，可认为是动画的「心脏」，功能类似于单片机的晶振。
每次计时脉冲，每一个动画都会根据动画的当前时间、Interpolator 和 TypeEvaluator，计算出动画的过程值。
更多内容，查阅 [http://li2.me/2016/01/android-animation-interpolator.html](http://li2.me/2016/01/android-animation-interpolator.html)


### 关于 animation frame
animation frame，at each frame，on each frame
动画帧（这里肯定不能把 frame 当做框架来理解）。

上述讲到 timing loop 时提到了计时脉冲，每次计时脉冲都会计算出一个新的动画值。有了新值，就意味着动画发生了变化，所以，可以理解为一帧一帧的动画。


### 关于 animated values
动画过程值；动画中间值。

属性动画的本质是「值的变化」，

- 通过构造器（或者 xml）指定动画的起止值；
- Interpolator 表征了「时间」的变化规律；
- TypeEvaluator 表征了「值」的变化规律；（「值」即对象的物质属性，或者是透明度，或者是背景色，或者是运动轨迹）

那么在整个动画的生命周期中，每个动画帧都会计算出一个值，这些值就是「animated values」，因此对象的「时」「空」就发生了变化，因此就有了动画。


### Interpolator 和 TypeEvaluator 的区别和联系
Interpolator 插值器：把处于某个区间（有起点值和终点值）的一个值，按照某种算法，映射为另一个值。

从上述定义的角度来看，Interpolator 和 TypeEvaluator 的作用是一样的。
**非要说区别的话，应该是新的动画机制对它俩的「职责」做了定性，从名字也能看出来端倪：一个管 time value（Interpolator 实现了 TimeInterpolator的接口），一个管 type value。**

具体到各自的方法：
Interpolator 的 `float getInterpolation(float fraction)` 虽然只有一个入口参数，是因为它的调用者已经把 @input 限定为 [0, 1.0]。而这个方法的作用是把动画的当前进度映射成另一个值，可谓是「篡改」，因此函数的名字「interpolation 篡改」起的非常合适。
而 TypeEvaluator 的 `T evaluate(float fraction, T startValue, T endValue)` 则是根据动画的初值、终值，和**被篡改的**播放进度，计算**动画过程值**，因此函数的名字「evaluate 评估」也是非常合适。

因此，从数学意义讲，二者有共性；从物理意义讲，二者有区别。
更多内容，查阅 [http://li2.me/2016/01/android-animation-interpolator.html](http://li2.me/2016/01/android-animation-interpolator.html)


### 关于 staggered animation
automatically staggered animation start times：自动错开动画的开始时间。

staggered adj. 错列的；吃惊的。 [LayoutAnimationController](http://developer.android.com/reference/android/view/animation/LayoutAnimationController.html) 用于实现容器布局动画，容器内的子视图动画相同，但开始时间不同。所以这里的 staggered 应是「错开的」意思。


### 其它一些单词和词组

- usher in 领进，引进。
- choreograph 设计舞蹈动作；为...编舞。
- choreograph multiple animations 为动画编舞；把多个动画优雅地编排成一个。（译注：真要好好学习动画，不辜负类名 Choreograph）。
- play with the API demos 和...玩耍。可以 play with 什么东西，小朋友之间也可以 play with，但是成人之间不要 play with 噢 (～﹃～)~zZ。
这里有一个歪果仁特意提到了： [中国人总是用错“play”这个单词，哈哈哈，我来解释一下](http://weibo.com/p/2304443ac2ae34008c494b295f9bdfc0b00c61)。


### 文中提到的一个哲学问题
[If a tree falls in a forest](https://en.wikipedia.org/wiki/If_a_tree_falls_in_a_forest)


### 相关博文

- [Android属性动画完全解析(上)，初识属性动画的基本用法 -- 郭霖的CSDN专栏](http://blog.csdn.net/guolin_blog/article/details/43536355)
- [Android属性动画完全解析(中)，初识属性动画的基本用法 -- 郭霖的CSDN专栏](http://blog.csdn.net/guolin_blog/article/details/43816093)
- [Honeycomb 中引入的新 Animation —— Property Animation](http://blog.csdn.net/zhaoshecsdn/article/details/46650163) （这是全文翻译）

------

**不把阅读英文文档当做翻译事业的程序员不是好厨子**
![da](/assets/img/android/android-animation-da-translation.png)
![er](/assets/img/android/android-animation-er-translation.png)

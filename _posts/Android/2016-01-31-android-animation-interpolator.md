---
layout: post
title: Android Animation Interpolator - Android 动画插值器源码笔记
category: Android
tags: [android-animation]
---


## 关于 [TimeInterpolator](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/animation/TimeInterpolator.java)

TimeInterpolator 是整个动画插值器的最顶层接口，只有一个方法，此处列出代码文件全文：

```java
package android.animation;

/**
 * A time interpolator defines the rate of change of an animation. This allows animations
 * to have non-linear motion, such as acceleration and deceleration.
 */
public interface TimeInterpolator {
    /**
     * Maps a value representing the elapsed fraction of an animation to a value that represents
     * the interpolated fraction. This interpolated value is then multiplied by the change in
     * value of an animation to derive the animated value at the current elapsed animation time.
     *
     * @param input A value between 0 and 1.0 indicating our current point
     *        in the animation where 0 represents the start and 1.0 represents
     *        the end
     * @return The interpolation value. This value can be more than 1.0 for
     *         interpolators which overshoot their targets, or less than 0 for
     *         interpolators that undershoot their targets.
     */
    float getInterpolation(float input);
}
```

从注释来看，它的作用是**「定义动画改变的速率，使得动画不一定要匀速改变，可以加速、减速。」**
真实世界不总是匀速运转的，如果我们针对不同的场景采用合适的插值器，动画的表现会自然好看，从而为 App 增添色彩 ( ˇˍˇ )

问题是，插值器是怎么办到的？继续看注释。
插值器用一个 0~1.0 范围的浮点数表示当前动画播放的进度，0 代表开始播放，1.0 代表播放结束。通过 `getInterpolation` **把当前进度映射成另一个值**。动画参照的时间由此被「篡改」，动画的速度由此被改变。

因此，我们可以这样理解插值器：把处于某个区间（有起点值和终点值）的一个值，按照某种算法，映射为另一个值。
Interpolator [汉语解释有两种](http://dict.youdao.com/search?le=eng&q=Interpolator&keyfrom=dict.top)：(1)【数学】内插器；分数计算器；(2)篡改者。印证了我们的理解。

接下来要思考的问题是，`getInterpolation` 是怎么被调用到的？要回答这个问题，我们必须回到 `ValueAnimator` 的源码：


## 关于 [AnimationHandler](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/animation/ValueAnimator.java#ValueAnimator.AnimationHandler)

AnimationHandler 是 ValueAnimator 定义的一个静态内部类，它实现了一个 runnable，在内部维护一个计时循环（和 `Choreographer` 有关 TODO），从而产生计时脉冲（timing pulse），动画之所以能「动」，就是计时脉冲在起作用，可认为是动画的「心脏」，功能类似于单片机的晶振。

程序内所有动画共享同一个脉冲，使所有动画共享一个时间，以保证同步。每次脉冲发生时，每一个动画都会根据当前时间（对应@input）和 TimeInterpolator（对应 getInterpolation方法），计算出另一时间（对应 @return）。以下是函数调用栈（不完全）：

```java
// 内部维护一个 runnable，`Choreographer` 应该是维护计时循环的核心类，暂不深究了 TODO；
run();

// 总之这个函数会被定时调用到，它遍历所有的动画，决定把某个动画放入等待队列，还是从等待队列中取出来执行；
doAnimationFrame(long mChoreographer.getFrameTime());

// 如果某个动画处于激活状态，则这个函数会被调用到；
anim.doAnimationFrame(long frameTime)

// 然后调用到这个函数，上述函数调用中的 frameTime 被转换成了 currentTime，表示动画的当前时间；
animationFrame(long currentTime);
    // 这个算式很简单，根据动画时长、当前时间、开始时间，算出动画的播放进度，一个 0~1.0 范围的浮点数；
    float fraction = mDuration > 0 ?
        (float)(currentTime - mStartTime) / mDuration : 1f;

// 接下来的这个函数调用就很关键了，它的入口参数是动画的播放进度；
animateValue(float fraction);
    // 其一：通过 Interpolator 篡改播放进度；
    fraction = mInterpolator.getInterpolation(float fraction);
    // 其二：通过 TypeEvaluator 计算动画过程值，PropertyValuesHolder 是处理计算的核心类 TODO；
    mValues[i].calculateValue(fraction);
    // 其三：回调监听器，发出动画值改变的通知；
    mUpdateListeners.get(i).onAnimationUpdate(this);
```


## 再次强调两个关键类：Interpolator 和 TypeEvaluator

Interpolator.getInterpolation(float) 篡改了播放进度；
TypeEvaluator 拿着被篡改的进度计算当前的动画过程值 `evaluate(float fraction, Object startValue, Object endValue)`。

这样，动画就可以忽快忽慢，或者全程加速，或者全程减速，或者是某种奇特的时间曲线。


## TimeInterpolator 的「徒子徒孙」

TimeInterpolator 毕竟只是一个接口，没有任何具体实现，所以播放进度到底是如何被篡改的呢？请看它的「徒子徒孙」：

```java
public interface TimeInterpolator {}
public interface Interpolator extends TimeInterpolator {}
abstract public class BaseInterpolator implements Interpolator {}
// 直接继承自 BaseInterpolator 的子类有：
AccelerateDecelerateInterpolator, AccelerateInterpolator, AnticipateInterpolator, AnticipateOvershootInterpolator, BounceInterpolator, CycleInterpolator, DecelerateInterpolator, LinearInterpolator, OvershootInterpolator, PathInterpolator
```

### [AccelerateDecelerateInterpolator](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/view/animation/AccelerateDecelerateInterpolator.java)

这是默认的插值器，如果 setInterpolator(null) 则是线性插值器。（[查阅API说明](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/animation/ValueAnimator.java#ValueAnimator.setInterpolator)）
它的行为：开始和结尾减速、中间加速。（PS：和绳命的轨迹相似（绳命的幼年和老年、绳命的青年），这是成为默认插值器的理由，一个猜想，可能是对的。）
它的插值方法：

```java
    public float getInterpolation(float input) {
        return (float)(Math.cos((input + 1) * Math.PI) / 2.0f) + 0.5f;
    }
```

什么？你说抽象？没关系，上图，插值方法对应的图形：

![AccelerateDecelerateInterpolator](/assets/img/android/android-animation-AccelerateDecelerateInterpolator.png)

红线是我特意标注的，在 [0, 1] 范围内，开头和结尾斜率小，中间斜率大。也就是说，两头慢，中间快。


### [CycleInterpolator](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/view/animation/CycleInterpolator.java)

CycleInterpolator 的插值方法：

```java
    public float getInterpolation(float input) {
        return (float)(Math.sin(2 * mCycles * Math.PI * input));
    }
```

CycleInterpolator mCycles=1 时的图形：

![CycleInterpolator](/assets/img/android/android-animation-CycleInterpolator.png)

被篡改后的播放进度为**负数**是怎么回事？需要回到 getInterpolation(float) 方法，查看它对返回值的解释：

> @return The interpolation value. This value can be more than 1.0 for
>         interpolators which overshoot their targets, or less than 0 for
>         interpolators that undershoot their targets.

[0, 1.0] 是未被篡改前的动画播放进度范围（即入口参数的范围），至于被篡改后的值，**超过 1 被称为「过冲」，小于 0 被称为「下冲」**。
过冲和下冲目前我理解的也不是很清楚（TODO），所以还是看 GIF 图吧：

CycleInterpolator 的 GIF 效果图：

![CycleInterpolator](/assets/img/android/android-animation-CycleInterpolator.gif)


### 从 ApiDemos 观察插值器的实际运行效果

**这里只列出了两个插值器，还有很多很多，理解这些插值器的最好办法是观看它们的运行效果，修改参数再次观看运行效果。** ApiDemos 包含这样的代码：ApiDemos/app/src/main/java/com/example/android/apis/view/Animation3.java

![interpolator](/assets/img/android/android-animation-interpolator-list.png)


## 附录

### interpolation 搜索自 CNKI 词典：

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

### Google graph

![google graph demo](/assets/img/util/google-graph-demo.png)

更多用法请查阅知乎问题 [如何高效地使用搜索引擎？](https://www.zhihu.com/question/28013848)

### 相关博文

[https://m.oschina.net/blog/137391](https://m.oschina.net/blog/137391)
[http://blog.kainaodong.com/?p=42](http://blog.kainaodong.com/?p=42)
[http://www.cnblogs.com/mengdd/p/3346003.html](http://www.cnblogs.com/mengdd/p/3346003.html)


### 张大锤「Interpolator」语录

当代生活太慢了！必须提速了！纵身一跃！旁友们，以光速投身在星辰大海，必须让灵魂数位化了！生活太缓慢简直是犯罪，你今天洗了几个碗。2014-03-29

人，不要闹腾，躺着最好。存在的真谛是慢，让生活慢下来。和乌龟学习！（三姨夫生活感悟 2014-07-26

历法完全是个骗局，纯粹是统治阶级为了mindfuck人们的工具，个体对时间流逝的感知不同，怎么能一样算呢，比如老张时间过的慢，今年只有20岁，但按公历算已经45了，不公平！小刘12岁按公历算已经32了，这样男的放进学校能放心吗？建议取消历法，让生活模糊起来！就分以前和以后，保持时间的连续性 2015-11-19

星期日下午三点半，你突然决定从此刻起比别人慢一秒，太棒了！在所有精确的严丝合缝的必须的不容置疑的社会主义生活方式中猛的后撤，过一种慢一步的生活，你成功了，一些不知所措在你上方集中，你陌生化成功了，生活第一次输了 2016-01-17

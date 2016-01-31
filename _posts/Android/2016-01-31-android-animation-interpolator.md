---
layout: post
title: Android Animation Interpolator - Android 动画插值器
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

从注释来看，它的作用是「定义动画改变的速率，使得动画不一定要匀速改变，可以加速、减速。」
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
它的行为：开始和结尾减速、中间加速。（PS：和绳命的轨迹相似（绳命的幼年和老年、绳命的青年），这是成为默认插值器的理由，一个猜测，可能是对的。）
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

它的插值方法：

```java
    public float getInterpolation(float input) {
        return (float)(Math.sin(2 * mCycles * Math.PI * input));
    }
```

插值方法对应的图形：

![CycleInterpolator](/assets/img/android/android-animation-CycleInterpolator.png)



> @return The interpolation value. This value can be more than 1.0 for
>         interpolators which overshoot their targets, or less than 0 for
>         interpolators that undershoot their targets.

![CycleInterpolator](/assets/img/android/android-animation-CycleInterpolator.gif)


## 

## 附录 interpolation
搜索自 CNKI 词典：
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


------

by
weiyi.li li2.me weiyi.just2@gmail.com
2016-01-28
禁止转载

---
layout: post
title: 如何在Android设备旋转时暂存数据以保护当前的交互状态?
category: Android
tags: [android-fragments, android-parcelable]
---

官方文档：[Handling Runtime Changes](http://developer.android.com/guide/topics/resources/runtime-changes.html)

## 设备旋转时保存Activity的交互状态

### 旋转时数据为什么会丢失？

**设备配置（device configuration）**用以描述设备当前状态，包括：屏幕方向、屏幕密度、屏幕尺寸、键盘类型、语言等。配置若在运行时发生变化（runtime configuration change），Android 会寻找更合适的资源以匹配设备配置。
**比如旋转设备会改变配置，那么 activity 实例会被系统销毁，然后创建一个新的 activity 实例。** 所以数据就丢掉了。

### 旋转时保存数据的方法

试想你在用手机看电子书，正读第21页呢，转了屏幕，竟从第1页开始了，会不会很恼火？会。
**因此需要采取某种方式保存数据，以便在旋转后恢复到旋转前的交互状态**。Android 考虑到了这种情况，提供了一个方法 `onSaveInstanceState()`，Android 会在杀掉 activity 前调用它，给了程序员一个机会，像这样；

```java
private static final String KEY_INDEX = "index";
private int mCurrentIndex = 0;

@Override
protected void onCreate(Bundle savedInstanceState) {
    if (savedInstanceState != null) {
        mCurrentIndex = savedInstanceState.getInt(KEY_INDEX, 0);
    }
}

@Override
protected void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    outState.putInt(KEY_INDEX, mCurrentIndex);
}
```

### onSaveInstanceState与onPause的区别

**要特别强调的是，onSaveInstanceState 并不是生命周期方法，Android 系统并不保证在杀死 activity 前一定调用它：**
当 activity 进入后台或者被销毁时 `onPause()` 总会被调到；当 activity 被销毁时 `onStop()` 总会被调到；而 `onSaveInstanceState()` 不会被调到的两种情况：

<!-- more -->

1. 当 activity B 被调到 activity A 上面，并且在B的生命周期内A未被杀掉，系统不会为A调用这个方法。因为A仍保持它原来的用户交互的状态。
2. 当用户按返回键从 activity B 返回到 activity A，系统不会为B调用这个方法。因为B的实例已被销毁，所以没有必要恢复状态。

**而严谨的你产生了疑问，既然onPause总会被调到，为什么不在onPause中保存数据，而要额外增加一个API做这件事情呢？**
**答案：要分清场合。**
程序员确实需要在onPause中保存数据（在onStop中做可能就来不及了），但是一般情况下保存的是永久性数据，比如preference、database、json file，可能还要给 thread、service 擦屁股；
而一些暂时的数据呢？比如上面讲的情况，只是旋转了屏幕，程序员就可以把当前页数放进 Bundle 中，交给 Application 保管。

### 旋转时方法的调用流程

```xml
旋转发生了，activity 实例被销毁：
onPause() // 存储永久数据
onSaveInstanceState(Bundle) // 存储暂时数据，在onStop前被调用
onStop()
onDestroy()

重建 activity 实例：
onCreate(Bundle) // 获取暂存数据
onStart()
onRestoreInstanceState(Bundle) // 也可在这获取暂存数据，在onStart后被调用
onResume() // 做对应于onPause的恢复的工作
```

此节内容参考：
> 部分内容翻译自 [andriod.app.Activity.onSaveInstanceState()源码注释](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/app/Activity.java)
> 部分内容整理自《Android权威编程指南3.2, 3.3》
> [QMLN31821007的CSDN专栏：Android中onSaveInstanceState(Bundle outState)与OnPause()的区别](http://blog.csdn.net/qmln31821007/article/details/39641983)

## 设备旋转时保存Fragment的交互状态

如上述，当设备旋转时，Activity实例将被销毁，那么附加在activity上的fragment也将被销毁。如果你在应用开发中喜欢使用 fragment 管理界面，碰巧某个案例中使用了Fragment管理音频播放，那么在旋转屏幕的时候，不中断音频的播放，体验是不是很棒？是。

虽然程序员可以覆写`onSaveInstanceState()`用来保存当前播放进度，但是播放会中断。怎么达成这个目的呢？
在Fragment的`onCreate()`方法中调用`setRetainInstance(true)`.
当旋转发生时，fragment实例会被保留（所有实例变量的值都保持不变），然后传递给新的activity，新的fragment manager依据保留的fragment重建它的视图。


对比上述看电子书的情形，假如电子书也是由fragment构建，采用setRetainInstance(true)的方式，也可以保证在旋转时当前页数同fragment一起被保留下来。
但是，被保留的fragment有可能会因系统回收内存而被销毁，那么页数也就丢失了。但是采用 `onSaveInstanceState(Bundle outState)` 的优势在于，bundle对象会保存的久一些（具体多久不清楚 TODO）。

如果Fragment只是简单的UI View，像是TextView, Button, CheckBox, ImageView... 不建议使用setRetainInstance方法，只需要记住当前fragment的index，然后在设备旋转后根据数据重新实例化一个fragment。因为不包含大量数据，旋转的过程中几乎可以用“无缝切换”来形容。
[一个相关问题](http://segmentfault.com/q/1010000003978225/a-1020000003978395)

**上述的讨论全是基于“设备旋转”的情况下，如何暂时地保存数据；如果Activity进入后台，程序员还是需要在onPause中长久地保存播放/阅读进度的。**



此节内容参考：

>  内容整理自《Android权威编程指南14》


## 设备旋转时保存WebView的数据

旋转设备屏幕时WebView将重新加载网页，这是因为WebView包含了太多的数据，以至无法在onSaveInstanceState(...) 方法内保存所有数据。

对于一些类似的类（如VideoView）， Android文档推荐让activity自己处理设备配置变更。也就是说，无需销毁重建activity可直接调整自己的视图以适应新的屏幕尺寸。这样， WebView也就不必重新加载全部数据了。不适用于所有视图（TODO不清楚哪些视图）。

```java    
<!-- Activity将自己处理设备配置的改变（键盘隐藏、屏幕方向改变、屏幕大小改）>
<activity android:name=".PhotoPageActivity"
    android:configChanges="keyboardHidden|orientation|screenSize" />
```    

此节内容参考：

>  内容整理自《Android权威编程指南31.3.2》

## 设备旋转时保存在自定义View中绘制的图形

[继承View以实现自定义View的绘制，比如矩形框](https://github.com/li2/Learning_Android_Programming/commit/a04ac1d7b699349b74ffbb7c2bf2ea579631c9af)
设备旋转后，绘制的矩形框会消失，要解决这个问题，可以使用以下View方法：

```java
protected Parcelable onSaveInstanceState();
protected void onRestoreInstanceState(Parcelable state);
```
`Parcelable onSaveInstanceState()` 建议保存的数据不应该是持久的或者稍后难以重建的。比如不应该保存屏幕的当前位置，因为位置会被重新计算。应该保存的，比如list view当前选中的条目。

与Activity和Fragment对应方法的区别是，代替Bundle参数，View的方法返回并处理的是实现了Parcelable接口的对象实例。
Parcelable接口允许类的实例可以被写入Parcel，并从中恢复，[官方文档示范了如何Parcelable化一个对象](
http://developer.android.com/intl/zh-cn/reference/android/os/Parcelable.html)。

### 可以向Parcel中写入什么呢？

- 可以把所有的元数据类型（Primitives）（byte, double, float, int, long, String）写入Parcel；
- 也可以把元数据的数组（Primitive Arrays）写入Parcel；
- 可以把Parcelable接口的对象写入Parcel；
- 可以把Bundle写入Parcel；
- 可以把Untyped Containers（Object, Object[], List）写入Parcel。

### 暂存ArrayList of Custom Objects的方法1

在自定义的View `BoxDrawingView`内定义一个继承自`BaseSavedState`的静态类`BoxDrawingState`，实现`writeToParcel(Parcel out, int flags)`方法，参考[github commit](https://github.com/li2/Learning_Android_Programming/commit/2d58e5e4ede7c0ff25b29e90db8dbdad3974dd7d)

```java
public class BoxDrawingView extends View {
    private ArrayList<Box> mBoxes = new ArrayList<Box>();
    
    static class BoxDrawingState extends BaseSavedState {
        ArrayList<Box> boxes;
        
        private BoxDrawingState(Parcel in) {
            super(in);
            in.readList(boxes, null);
        }
        
        @Override
        public void writeToParcel(Parcel out, int flags) {
            super.writeToParcel(out, flags);
            out.writeList(boxes);
        }
    }
    
    @Override
    protected Parcelable onSaveInstanceState() {
        Parcelable superState = super.onSaveInstanceState();
        BoxDrawingState boxDrawingState = new BoxDrawingState(superState);
        boxDrawingState.boxes = mBoxes;
        return boxDrawingState;
    }
    
    @Override
    protected void onRestoreInstanceState(Parcelable state) {
        BoxDrawingState boxDrawingState = (BoxDrawingState) state;
        super.onRestoreInstanceState(boxDrawingState.getSuperState());
        mBoxes = boxDrawingState.boxes;
    }
}
```

### 暂存ArrayList of Custom Objects的方法2

另一种暂存ArrayList of Custom Objects的方法，Parcelable化自定义类，调用`Bundle.putParcelableArrayList(String key, ArrayList<? extends Parcelable> value)`. [参考 github commit](https://github.com/li2/Learning_Android_Programming/commit/47bc602622e4f55deef16acb37b24f20162f7b63)


此节内容参考：
>  内容整理自《Android权威编程指南32.5》
> [How to prevent custom views from losing state across screen orientation changes-方法1](http://stackoverflow.com/a/3542895/2722270)
> [How to prevent custom views from losing state across screen orientation change-方法2](http://stackoverflow.com/a/8127813/2722270)
> [Android View onSaveInstanceState not called](http://stackoverflow.com/a/28586444/2722270)



## 设备旋转时，如何处理正在运行的线程？ TODO



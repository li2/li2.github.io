---
layout: post
title: 「Android编程权威指南笔记」Activity
category: Android
tags: [ ]
---

## 第2章 Android与MVC模式
如何创建一个新类，并自动为成员变量生成getter和setter方法，以及成员变量命名的约定；
android命名规范，m作为Fields前缀，s作为Static Fields前缀。
eclipse可以自动生成getter和setter方法。

介绍MVC模式，使用MVC设计软件的优点；
使用MVC扩展了第1章的范例：
更新View；
更新Controller及模型（涉及到的编程原则，封装公共代码）；
介绍了另一种资源文件：图片资源文件，添加和引用的方法。
扩展练习：使用ImageButton、为TextView添加监听器、隐藏的与activity lifecycle有关的bug。

<!-- more -->

## 第4章 android应用的调试
如何处理应用bug. 如何使用LogCat、Android Lint和Eclipse内置的代码调试器。
DDMS透视图（ Dalvik Debug Monitor Service，调试监控服务工具），包含LogCat以及Devices视图。
可以查看异常及其**栈追踪(stack trace)**（应该是指方法的调用信息）. 
一般情况下，在LogCat中寻找最后一个异常及其栈追踪信息的第一行（该行记录发生异常的类、方法、源文件、代码行号）。
对于非崩溃型的异常（比如代码逻辑错误），只能在可能导致问题的地方：

- 主动抛出异常，用于查看某个方法的栈追踪；
- 利用调试器设置合适的断点。


## 监听器
Android应用属于典型的事件驱动类型。不同于命令行或脚本程序,事件驱动型应用启动后,即开始等待行为事件的发生,如用户单击某个按钮。(事件也可以由操作系统或其他应用触发,但用户触发的事件更显而易见。)
应用等待某个特定事件的发生,也可以说该应用正在“监听”特定事件。为响应某个事件而创建的对象叫做监听器(listener)。监听器是实现特定监听器接口的对象,用来监听某类事件的发生。
监听器需实现 View.OnClickListener 接口。

```Java
mButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
    }
});
```
通过匿名内部类实现监听器，代码块简洁、减少命名类的使用。


## 第3章 Activity的生命周期

Activity管理用户和屏幕的交互。
当Activity子类的实例创建后，onCreate()方法会被调用，需要获取并管理属于它的界面。通过传入布局的资源ID参数，该方法生成指定布局的视图并将其放置在屏幕上。布局视图生成后，布局文件包含的组件也随之以各自的属性定义完成实例化。

```Java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
}
```

每个activity实例都有生命周期。在生命周期内，activity在运行、暂停、停止3种状态间切换，状态切换时系统会调用生命周期方法，所以需要override生命周期方法，在状态切换时完成状态保存/恢复之类的工作。

override生命周期方法时，必须通过super关键字调用超类的方法。onCreate()必须先调用超类方法。
可以通过日志跟踪生命周期。通过logcat工具查看日志。
启动activity时，onCreate() -> onStart() -> onResume();
单击设备后退键（相当于通知系统“我已经完成activity的使用，现在不需要它了”）：onPause() -> onStop() -> onDestroy();
单击设备Home键（相当于通知系统“我去别处看看，稍后可能会回来”）：onPause() -> onStop(). 此时，为了快速响应随时返回应用，系统只是暂停当前activity，并不销毁它。但是系统不保证暂停的activity常驻内存，当系统需要回收内存时，将首先销毁暂停的activity.

### 设备旋转时activity生命周期的变化？

涉及设备配置、备选资源的概念，layout的备选资源，framelayout组件位置排列？
设备旋转前如何保存数据？覆盖onSaveInstanceState()方法。
称之为“暂存状态”。常见的做法？
系统settings -> Development Options -> Don't keep activities选项改变了？

### 设备旋转前保存数据

设备旋转时，当前的activity实例会被系统销毁，然后创建一个新的实例。所以需要在设备旋转时保存数据，使得可以恢复到旋转前的状态。
一种解决方法是覆写(override) activity方法：`protected void onSaveInstanceState(Bundle outState)`. 在`onPause()`, `onStop()`, `onDestroy()`方法之前由系统调用。要求所有activity的视图将自身状态数据保存在Bundle对象中。Bundle是存储字符串键与限定类型值之间映射关系（键值对）的一种结构。

```Java
private static final String KEY_INDEX = "index";
private int mCurrentIndex = 0;

@Override
public void onSaveInstanceState(Bundle savedInstanceState) {
    super.onSaveInstanceState(savedInstanceState);
    savedInstanceState.putInt(KEY_INDEX, mCurrentIndex);
}

在onCreate()方法中检查存储的bundle信息：
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    if (savedInstanceState != null) {
       mCurrentIndex = savedInstanceState.getInt(KEY_INDEX, 0);
    }
}
```
我们在Bundle中存储和恢复的数据类型只能是基本数据类型（primitive type）以及可以实现Serializable接口的对象。创建自己的定制类时，如需在onSaveInstanceState(...)方法中保存类对象，记得实现Serializable接口。

### Activity会被销毁的情况：

- 旋转屏幕；
- 用户离开当前界面；
- Android需要回收内存时（只有暂停或停止状态才可能被回收）；

bundle被存储于activity的**ActivityRecord**.
ActivityRecord可以保留多久？用户按了Back Button时，系统会彻底销毁当前的activity，记录同时被清除。此外，系统重启或者长时间不使用activity时，记录通常也会被清除。


## 第5章 创建新的Activity

创建新的activity，需要

- 创建布局文件（以及可能用到的资源文件）；
- 创建新的activity子类；
- 在manifest配置文件中声明activity；

manifest配置文件是一个包含元数据的XML文件，用来向Android操作系统描述应用。该文件总是以AndroidManifest.xml命名。
应用的所有activity都必须在manifest配置文件中声明，这样操作系统才能够使用它们。

### 启动Activity

Intent对象是组件(component)用来与操作系统通信的一种媒介工具。Intent类提供了多个构造方法，以满足不同组件的需求。对于activity组件：
`public Intent(Context packageContext, Class<?> cls)`
Class对象是需要启动的activity，Context对象指明Class对象所在的包。

```Java
Intent i = new Intent(CurrentActivity.this, DestActivity.this);
startActivity(i);
```
这是显示的(Explicit)intent.

### Activity间传递数据

Intent提供`putExtra`方法，可以在activity间传递数据。extra是key-value结构，可以是任意数据。

```Java
public static final String EXTRA_ANSWER_IS_TRUE = "packagename.answer_is_true";

Intent i = new Intent(CurrentActivity.this, DestActivity.this);
boolean answer_is_true = true;
i.putExtra(CurrentActivity.EXTRA_YOUR_DATA, answer_is_true);
startActivity(i);
```

从extra获取数据的方法，如果是boolean变量：

```Java
private boolean mAnswerIsTrue;

@Override
public void onCreate(Bundle savedInstanceState) {
    ......
    mAnswerIsTrue = getIntent.getBooleanExtra(EXTRA_ANSWER_IS_TRUE, false);
}
```

### 从子activity获取返回结果

#### 父Activity应该这样实现：

```Java
Intent i = new Intent(CurrentActivity.this, DestActivity.this);
// 启动多个child activity时，用来区分返回的结果属于哪个child activity
int requestCode = 0;
startActivityForResult(i, requestCode);

// 覆写onActivityResult()方法以处理返回结果
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent intent) {
    if (intent == null) {
        return;
    }

    // 从Intent中获取返回数据
    if (requestCode == 0) {

    }
}
```

#### 子Activity应该这样实现:
数result code可以是以下两个预定义常量中的任何一个, `Activity.RESULT_OK`, `Activity.RESULT_CANCELED`, （如需自己定义结果代码，还可使用另一个常量：RESULT_FIRST_USER）。
在父activity需要依据子activity的完成结果采取不同操作时，设置结果代码很有帮助，比如子activity有一个OK按钮及一个Cancel按钮。

```Java
Intent intent = new Intent();
int resultCode = RESULT_OK;
setResult(resultCode, intent);
```

在桌面点击app图标时，os只是启动了系统的一个activity，即launcher activity。使用向导创建的activity默认被设置为launcher activity，在manifest中的定义：

```xml
<activity
    android:name=""
    android:label="@string/app_name" >
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

---

整理笔记时参考了如下资料：

- 《Android编程权威指南》Bill Phillips  Brian Hardy著，王明发 译。人民邮电第1版。
    英文版书名《Android Programming - The Big Nerd Ranch Guide》

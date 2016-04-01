---
layout: post
title: 「App Training笔记」创建第一个应用
category: Android
tags: [android-layout]
---
 
[原文地址 http://developer.android.com/training/basics/firstapp/index.html](http://developer.android.com/training/basics/firstapp/index.html)
演示如何搭建开发环境。
演示如何创建一个android工程，运行一个可调式的app。学习app设计的一些基本要素，包括：创建简单的用户接口、处理用户输入。
笔记整理于2014-09-03 18:40 by li2

<!-- more -->

## 使用eclipse创建工程
create a Project with Eclipse
 
演示了如何使用android sdk tools的向导创建工程。需要注意的是如下概念的区别：Application Name, Project Name, Package Name, Minimum Required SDK, Target SDK, Compile With.

## 运行你的应用
Running Your App
 
介绍了android工程中的几个路径和文件： AndroidManifest.xml, src/, res/, drawable-hdpi/, layout/, values/.
演示了如何在android设备上安装、运行app。

## 建立简单的用户接口
Building a Simple User Interface
 
android图形用户界面是使用**View和ViewGroup层级结构（hierarchy objects）**创建的。View objects是常用的**窗口部件**，比如按钮和文本框。ViewGroup objects是 ** 定义child views布局的可视化容器**，比如网格部件（grid）或者垂直列表部件（vertical list）。
android提供了对应于View和ViewGroup子类的xml词汇表（vocabulary？），你可以在XML文件中使用UI元素的层级结构创建自己的UI。
 
演示了如何创建一个包含按钮和文本框的布局文件。
在xml中声明UI布局，要比在代码中好，因为你可以为不同的屏幕创建相应的布局，并告诉系统如何选择。
 
这里提到了若干概念：元素（element）、属性（attribute）。

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"           //这两个属性指定LinearLayout的宽度
    android:layout_height="match_parent"          //和高度：充满整个屏幕。
    android:orientation="horizontal" >            //以水平/垂直方式布局child views。

    <EditText android:id="@+id/edit_message"      //供代码引用的惟一标识符，读改view，+号仅用于定义ID。
        android:layout_weight="1"                 //权重，用来指定可以占多大比例的的剩余空间。
        android:layout_width="0dp"                //按照内容大小指定宽度和高度。
        android:layout_height="wrap_content"
        android:hint="@string/edit_message" />    //

</LinearLayout>                                   //LinearLayout是ViewGroup的子类，是线性列表部件。
```
 
资源属性一般定义成如下形式：`"@ResourceType/ResourceName"`
at符号（`@`）、资源类型（resource type）、斜杠slash(`/`)、资源名称（resource name）。比如`"@string/edit_message"`（使用资源文件，而不是硬编码（hard-coded）字符串，使得你可以在同一个地方维护UI文本，这种方式还可以支持多国语言。）
 
这种定义资源属性的方式，是在资源类型界定的范围内， 引用具体的资源文件（不同于id，在编译app时由sdk创建，这也是id需要+号的原因），所以id和hint使用了相同的资源名称也不会引起冲突。
 
一个资源对象（resource objects）仅仅是一个惟一的整数名字，关联到一个app资源，比如图片、布局文件、字符串。
每种资源都有相应的资源对象，定义在工程的gen/R.java（编译时由sdk生成，不能手动修改）中。使用类R中的对象名称来引用app资源。


## 启动另一个Activity
Starting Another Activity
 
**Intent**是一个对象，在运行时绑定不同的组件（比如2个activity）。Intent表示一个app “intent to do something”。可以完成多种任务，常用于**启动另一个activity，并且可以传递数据**。
 
API原型，需要导入Intent类：

```java
import android.content.Intent;
Intent(Context packageContext, Class<?> cls)
putExtra(String name, String value)
```
 
一个按钮的响应函数（`android:onClick="sendMessage"`），用来启动另一个activity，并发送字符串：

```java 
public final static String EXTRA_MESSAGE = "com.example.myfirstapp.MESSAGE";        
/** Called when the user clicks the Send button */
public void sendMessage(View view) {
    //Context是第一个参数，因为Activity类是Context的子类，所以可以使用this
    //Class是第二个参数，表示app组件的类，系统会把Intent应用到该类。（此处的Intent是启动另一个activity）
    Intent intent = new Intent(this, DisplayMessageActivity.class);
    EditText editText = (EditText) findViewById(R.id.edit_message);
    String message = editText.getText().toString();

    //Intent通过 key-value 形式携带不同类型的数据，称之为extras，
    //为了让另一个activity查询extras，需要定义key为public constant，最好以app‘s package name作为前缀。
    intent.putExtra(EXTRA_MESSAGE, message);

    //to start an activity, and pass it your Intent.
    startActivity(intent);
}
```
 
此例的intent是显示的（explicit），因为明确指出了intent应用到的app组件。intent也可是隐式的（implicit），因为没有指出intent应用到的组件，但允许设备上的app响应该intent，只要app满足Intent参数所定义行动的metadata规格。
 
**通过eclipse创建activity - Create the Second Activity**
（而非手动创建一个以.java为后缀的文件），会自动创建相应的java文件（已生成onCreate()等方法）、layout文件、向string资源文件中添加字符串定义、向manifest文件中添加activity声明。
 
**从Intent中取回数据 - Receive the Intent**
 
```java 
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // Get the message from the intent
    Intent intent = getIntent();
    String message = intent.getStringExtra(MainActivity.EXTRA_MESSAGE);

    // Create the text view，这种方法是在代码中动态定义，而非在xml中事先声明
    TextView textView = new TextView(this);
    textView.setTextSize(40);
    textView.setText(message);

    // Set the text view as root view of the activity's layout
    setContentView(textView);
}
```

**至此，已经创建了属于你的第一个app。**

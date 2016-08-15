---
layout: post
title: Make a Reusable UI in Android App Development
description: 
category: Android
tags: [android-view]
---

It's important to an App developer to build a reusable, flexible and easy-to-modify UI when develop an Android App. You might imagine that you are developing an App and most of its activities have almost the same layout, like this:

- The top toolbar bar with a back button, a title view and a close button.
- The main part of the screen which shows a list view, or a detial view, or a camera preview.
- The bottom toolbar with two buttons which perform certain operations when clicked.

![](/assets/img/android/make-a-reusable-ui.png)

Using UI fragments separates the UI of your app into building blocks, and create an abstract activity for hosting a fragment.


*Copyright: [Make a Reusable UI in Android App Development](http://li2.me/2016/08/make-a-reusable-ui-in-android-app-development.html) is written by Weiyi.Li [http://li2.me](http://li2.me)*



## Step1: Create A Generic Fragment-Hosting Layout

- To efficiently re-use complete layouts, you can use the [\<include/\> tag](https://developer.android.com/training/improving-layouts/reusing-layouts.html) to embed another layout inside the current layout.
- To make a spot for the fragment's view in the activity's view hierarchy, you can use `FrameLayout` as a container view for hosting fragment;
- To lazily inflate layout resources at runtime, you can use `ViewStub`.

Code Snippet:
<script src="https://gist.github.com/li2/f4035ac857abf5318ce30cae1b5ac247.js"></script>

`toolbar_actionbar.xml` code [click here](https://gist.github.com/38794f9afb5047579f32dc42aca340dd.git).



## Step2: Create An Abstract Activity class

Nearly every activity you will create in this app will require the same code:

- Set the ActionBar title, callbacks when ActionBar's Back & Close button clicked.
- Create a fragment instance and commit it.

To avoid typing it again and again, you are going to stash it in an abstract class `BasicActivity.java`, code snippet::
<script src="https://gist.github.com/li2/71990eec8eea0edcc483eafd6083a0f0.js"></script>

To inflate bottom toolbar layout for activity which has a bottom toolbar, create another abstract class `BasicOperationActivity.java` which extends `BasicActivity.java`, code snippet:
<script src="https://gist.github.com/li2/2de13d883b02f2fdbdc712183c035d29.js"></script>



## Step3: Create A Single Fragment Activtiy

Then you will create a single fragment activity, code snippet:
<script src="https://gist.github.com/li2/d9c37cb86b5ba7d2ce0bdc9e0e067c0f.js"></script>


## Step4: Create Two Fragments Activity

If the activity has more than one fragment, you will use `show` & `hide` methods to manager these fragments, code snippet:
<script src="https://gist.github.com/li2/f682d2f6ef6e4b1316b7722fb7eedfd5.js"></script>

The End.

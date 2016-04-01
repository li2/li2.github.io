---
layout: post
title: 通过jfeinstein10/SlidingMenu实现Android侧滑菜单
category: Android
tags: [android-fragments]
---

[在stackoverflow上的回答](http://stackoverflow.com/a/29326687/2722270)

If you use [jfeinstein10/SlidingMenu](https://github.com/jfeinstein10/SlidingMenu) in your project, you can implement sliding menu in your `FragmentActivity`, **as you want**.

## step1. new SlidingMenu and attach to your activity.

Refer to [How to Integrate this Library into Your Projects](https://github.com/jfeinstein10/SlidingMenu#how-to-integrate-this-library-into-your-projects):

> You can wrap your Activities in a SlidingMenu by constructing it programmatically (
>
>`menu = new SlidingMenu(Context context)`) and then calling
>
> `menu.attachToActivity(Activity activity, SlidingMenu.SLIDING_WINDOW | SlidingMenu.SLIDING_CONTENT)`.
>
> `SLIDING_WINDOW` will include the Title/ActionBar in the content section of the SlidingMenu, while
> `SLIDING_CONTENT` does not. 
> You can check it out in the example app **AttachExample Activity**.

## step2. Then implement a `SampleListFragment` which represent your menu, and add to your activity.

define a fragment container layout `fragment_menu.xml`:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/fragmentContainer"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
```

call `menu.setMenu(R.layout.fragment_menu);` **to add the fragment container view to your activity**,
Lastly, **add your `SampleListFragment` to your activity's fragment manager**:

```java
		getSupportFragmentManager()
		.beginTransaction()
		.replace(R.id.fragmentContainer, new SampleListFragment())
		.commit();
```

------

In this way, you got a sliding menu in your fragment activity.
That's all.

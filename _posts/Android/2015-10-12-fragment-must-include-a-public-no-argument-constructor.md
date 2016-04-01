---
layout: post
title: Fragment子类必须包含一个public无参构造器
category: Android
tags: [android-fragments]
---

碰到一个很难复现的crash，LogCat捕捉到如下crash信息：

> android.support.v4.app.Fragment$InstantiationException: Unable to instantiate fragment MyFragment: make sure class name exists, is public, and has an empty constructor that is public

从Fragment文档中看到可能与此crash相关的说明：
译：所有Fragment子类必须要包含无参数构造器。Framework在必要时候常常需要重新实例化一个fragment，尤其在恢复fragment状态时，需要能找到这个构造器。如果找不到，在上述提到的情况下就会抛出运行时异常。

> 原文：All subclasses of Fragment must include a public no-argument constructor. The framework will often re-instantiate a fragment class when needed, in particular during state restore, and needs to be able to find this constructor to instantiate it. If the no-argument constructor is not available, a runtime exception will occur in some cases during state restore.

因为我偏向于使用如下方法实例化Fragment，所以代码中就没再定义构造器：

```java
public static MyFragment newInstance(Date date) {
    Log.d(TAG, "newInstance");
    Bundle args = new Bundle();
    args.putLong(EXTRA_DATE, date.getTime());

    MyFragment fragment = new MyFragment();
    fragment.setArguments(args);
    return fragment;
}
```

为了解决crash，以及保险起见，就添加了一个空的构造器：

<!-- more -->

```java
public MyFragment() {
    super();
}
```

因为crash很难复现，所以并不能确定是否有用。

参考：
[Fragment Class Overview](https://developer.android.com/intl/zh-cn/reference/android/app/Fragment.html)
[Android InstantiationException With Fragment (It Is Public)](http://stackoverflow.com/questions/14031110/android-instantiationexception-with-fragment-it-is-public)

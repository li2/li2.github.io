---
layout: post
title: 如何获取FragmentTabHost中指定标签页的Fragment？
category: Android
tags: [android-fragments]
---

使用FragmentTabHost构建了包含几个标签页的界面，如何获取指定标签页的Fragment？
How to get Fragment in FragmentTabHost?

## TabHost中Fragment的Tag

一般通过`FragmentTabHost.addTab(TabSpec tabSpec, Class<?> clss, Bundle args)`方法添加fragment：

```java
TabSpec tabSpec = mTabHost.newTabSpec("TAB_TAG_" + i).setIndicator("TAB_TITLE_" + i); 
mTabHost.addTab(tabSpec, MyFragment.class, null);    
```
而方法`TabHost.TabSpec newTabSpec(String tag)`的参数`tag`**就是fragment的tag**.
那么，我们有2种方法获取fragment。


## 通过`Fragment.getTag()`获取Fragment

最开始采用了这种办法，首先获取fragment manager管理的fragment列表，然后根据tag从列表中查找fragment：

<!-- more -->

```java
private Fragment getFragment(int tabId)
{
    List<Fragment> fragments = getSupportFragmentManager().getFragments();
    for(Fragment fragment : fragments) {
        String str1 = fragment.getTag();
        String str2 = String.valueOf("TAB_TAG_" + tabId);
        if(str1 != null && str1.equals(str2)) // 最开始没有检查str1是否为空，导致crash！
            return fragment;
    }
    return null;
}
```

然后掉进坑里了！
因为拿到的fragment列表，不仅仅是TabHost包含的fragment，**还包含向attached activity添加的其它fragment，而如果这些fragment并未设置tag，那么fragment.getTag()将返回null，然后就crash了**。


## 通过`FragmentManager.findFragmentByTag(String tag)`获取Fragment

感觉这个方法最简洁。

```java
private Fragment getFragment(int tabId)
{
    return getSupportFragmentManager().findFragmentByTag("TAB_TAG_" + tabId);
}
```

## 在`onTabChanged()`中获取的fragment有时为空

打印了一些log发现，

- 如果第1次切换到某个标签页，在`onTabChanged()`方法中立刻调用`findFragmentByTag()`时，返回的总是null，但如果延迟一段时间，就一定可以获取fragment.
- 如果再次切换到某个标签页，在`onTabChanged()`中不需要延时，总可以返回fragment.

```java
@Override
public void onTabChanged(final String tabId)
{
    Fragment fg = getSupportFragmentManager().findFragmentByTag(tabId);
    Log.d(TAG, "onTabChanged(): " + tabId + ", fragment " + fg);

    if (fg == null) {
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                Fragment fg = getSupportFragmentManager().findFragmentByTag(tabId);
                Log.d(TAG, "onTabChanged() delay 50ms: " + tabId + ", fragment " + fg);
            }
        }, 50);
    }
}
```

LogCat输出:

```xml
// cannot get the selected fragment immediately if the fragment has never been instantiated.
onTabChanged(): 1, fragment null
onTabChanged() delay 50ms: 1, fragment HistoryFragment{6f7a9d5 #2 id=0x7f09006e 1}
onTabChanged(): 2, fragment null
onTabChanged() delay 50ms: 2, fragment HistoryFragment{10c59e72 #3 id=0x7f09006e 2}

// can get the selected fragment immediately if the fragment already instantiated.
onTabChanged(): 1, fragment HistoryFragment{6f7a9d5 #2 id=0x7f09006e 1}
onTabChanged(): 2, fragment HistoryFragment{10c59e72 #3 id=0x7f09006e 2}
```

[How to get current selected Fragment in FragmentTabHost](http://stackoverflow.com/questions/27854072/how-to-get-current-selected-fragment-in-fragmenttabhost/33028970#33028970)

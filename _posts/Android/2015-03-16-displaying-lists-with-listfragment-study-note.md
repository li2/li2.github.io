---
layout: post
title: 「Android编程权威指南笔记」使用ListFragment显示列表
category: Android
tags: [android-fragments]
---

使用 ListFragment(Fragment 的子类) 构建列表视图，

## 使用 ArrayList<E> 和 Singleton 构建应用的模型层

ArrayList<E>是一个支持存放指定数据类型对象的Java有序数组类，具有获取、新增及删除数组中元素的方法。
把列表需要显示的数据集中存储(centralized data storage)在一个单例(Singleton)中。单例是特殊的Java类，一个类只允许创建一个实例。

### 创建单例
应用能够在内存里存在多久,单例就能存在多久,因此将对象列表保存在单例里可保持crime数据的一直存在,不管activity、fragment及它们的生命周期发生什么变化。
需创建一个带有私有构造方法及get()方法的类，其中get()方法返回实例。如实例已存在，get()方法则直接返回它；如实例还未存在，get()方法会调用构造方法来创建它。

```Java
// Setting up the Singleton
public class CrimeLab {
    private static CrimeLab sCrimeLab;	// 静态变量以s前缀标示
    private Context mAppContext;

    private CrimeLab(Context appContext) {
        mAppContext = appContext;
    }

    public static CrimeLab get(Context c) {
        if (sCrimeLab == null) {
            sCrimeLab = new CrimeLab(c.getApplicationContext);
        }
        return sCrimeLab;
    }
}
```

CrimeLab 类的构造方法需要一个 Context 参数。使用 Context参数,单例可完成启动activity、获取项目资源,查找应用的私有存储空间等任务。Context可能是一个 Activity ,也可能是另一个 Context 对象,如 Service 。为保证单例总是有 Context 可以使用,可调用 getApplicationContext() 方法，是针对应用的全局性Context。任何时候,只要是应用层面的单例,就应该一直使用application context。


## 创建 ListFragment
ListFragment 继承自 Fragment, 默认具有一个全屏的 ListView 布局。它的`onCreate()`方法继承自`Fragment`.

<!-- more -->

```Java
// CrimeListFragment.java
public class CrimeListFragment extends ListFragment {
    private ArrayList<Crime> mCrimes;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        getActivity().setTitle(R.string.crimes_title);
        mCrimes = CrimeLab.get(getActivity()).getCrimes();
    }
}
```

## 使用抽象 Activity 托管 Fragment
接下来需要创建 Fragment 容器布局来托管 Fragment（之前的章节已经了解到 Fragment 不具备在屏幕上显示视图的能力）。
因为容器并未指定特定的 Fragment, 所以只要一个 Activity 托管一个 Fragment 就可以使用该布局文件。

```xml
// 通用的 Fragment 容器布局文件 activity_fragment.xml
<FragmeLayout xmlns:android:"https://schemas.android.com/apk/res/android"
  android:id="@+id/fragmentContainer"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  />
```

在这种情形下：**每创建一个 Activity 就需要托管一个 Fragment**，我们可以将这样的 Activity 抽象出来，以避免代码的重复输入。

### 抽象托管 Fragment 的 Activity类

```Java
public abstract class SingleFragmentActivity extends FragmentActivity {
    // 由子类实现的抽象方法，创建新的 Fragment 实例
    protected abstract Fragment createFragment();

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_fragment);
        FragmentManager fm = getSupportFragmentManager();
        Fragment fragment = fm.findFragmentById(R.id.fragmentContainer);
        if (fragment == null) {
            fragment = new createFragment();
            fm.beginTransaction()
                .add(R.id.fragmentContainer, fragment)
                .commit();
        }
    }
}

// CrimeActivity.java
public class CrimeActivity extends SingleFragmentActivity {
    @Override
    protected Fragment createFragment() {
        return new CrimeListFragment();
    }
}
```

## ListFragment、ListView 及 ArrayAdapter
ListView 的每一个列表项，是一个View的子对象，既可复杂，也可简单。
ListView在需要显示的时候才创建视图对象，向 Adapter 申请。
Adapter 是控制器对象，**从模型层获取数据，用数据填充视图对象，并将准备好的视图对象提供给ListView显示**。
![image](/assets/img/android/图9-8-ListView-Adapter会话.png)
`ArrayAdapter.getCount()`返回视图对象个数；`ArrayAdapter.getView()`构建视图对象并返回；ListView将其设置为子视图。

### 创建 ListView 的 ArrayAdapter<T> 实例

```Java
// CrimeListFragment.java
ArrayAdapter<Crime> adapter = new ArrayAdapter<Crime>(
    getActivity(), // Context 对象
    android.R.layout.simple_list_item_1, // Android SDK提供的布局资源
    mCrimes); // 数据集
// 类ListFragment的方法，为内置 ListView 设置 Adapter.
setListAdapter(adapter);
```

### 响应列表项的点击事件
需要覆盖 ListFragment 的方法：
`public void onListItemClick(ListView l, View v, int position, long id)`

## 定制 ListItem
SDK提供的布局资源`layout.simple_list_item_1`仅包含一个`TextView`，我们需要更复杂的列表项，因此要定制化，需要2步：

- 创建定义列表项视图的xml布局文件；
- 创建 ArrayAdapter<T>的子类，用来创建、填充、返回新布局文件所定义的视图。

### 创建 ListView 的 ArrayAdapter<T> 子类
`convertView`是一个已经存在的列表项，adapter可重新配置并返回它。复用视图对象可以避免反复创建、销毁同一类对象的开销，提升应用性能。
调用 `Adapter.getItem(int)`方法获取数据集里position位置的Crime对象。
`ListView.getListAdapter()`方法可以获取绑定到ListView的adapter。

```Java
// CrimeListFragment.java
private class CrimeAdapter extends ArrayAdapter<Crime> {
    public CrimeAdapter(ArrayList<Crime> crimes) {
        super(getActivity(), 0, crimes);
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        // If we weren't given a view, inflate one
        if (convertView == null) {
            convertView = getActivity().getLayoutInflater()
                .inflate(R.layout.list_item_crime, null);
        }

        // Get data        
        Crime c = getItem(position);

        // Configure the view for this Crime
        TextView titleView = (TextView) convertView.findViewById(...);
        titleView.setText(c.getTitle());
        ......

        return convertView;	
    }
}
```
然后需要绑定adapter到 ListView，更新`onCreate()`和`onListItemClick()`方法，以使用 CrimeAdapter，代替 ArrayAdapter。

---

整理笔记时参考了如下资料：

- 《Android编程权威指南》Bill Phillips  Brian Hardy著，王明发 译。人民邮电第1版。
    英文版书名《Android Programming - The Big Nerd Ranch Guide》

---
layout: post
title: 「Android编程权威指南笔记」UI Fragment
category: Android
tags: [android-fragments]
---

## 第7章 UI Fragment与FragmentManager

**UI Fragment可以管理界面，整屏或部分。有自己的布局文件，包含了用户可以交互的可视化UI元素。**
用UI Fragment将应用的UI分解成块，利用一个个构建块，很容易做到构建分页界面、动画侧边栏界面等更多其他定制界面。
Fragment不具有在屏幕上显示视图的能力。因此，只有将它的视图放置在activity的视图层级结构中（称之为**托管UI Fragment**），fragment视图才能显示在屏幕上。

在activity代码中添加fragment, 可以在运行时控制fragment,我们可以决定何时将fragment添加到activity中以及随后可以完成何种具体任务；也可以移除fragment，用其他fragment代替当前fragment，然后再重新添加已移除的fragment。

### 创建Fragment容器布局

首先需要在activity视图层级结构中为fragment视图安排位置，**创建fragment容器布局**：

<!-- more -->

```xml
activity_crime.xml
<FragmeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragmentContainer"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    />
```
但此时，activity还未托管fragment，所以代码运行后UI看不到任何内容。接下来需要编写代码，**创建UI Fragment**，覆写fragment的生命周期函数（几乎对应到activity的声明周期函数）。

### 创建UI Fragment

创建fragment和创建activity步骤相同：定义布局文件、创建fragment子类、在代码中关联布局文件声明的组件。

```Java
public class CrimeFragment extends Fragment {
    private Crime mCrime;
    private EditText mTitleField;
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mCrime = new Crime();
    }

    @Override
    // 由onCreateView方法生成fragment的视图
    public View onCreateView(LayoutInflater inflater, ViewGroup parent, Bundle savedInstanceState) {
        View v = inflater.inflate(R.layout.fragment_crime, parent, false);
        // 调用View.findViewById(int)
        mTitleField = (EditText)v.findViewById(R.id.crime_title);
        // 监听器方法设置和activity一样
        mTitleField.addTextChangedListener(new TextWathcer() {
	    public void onTextChanged(...) {}
	    public void beforeTextChanged(...) {}
	    public void afterTextChanged(...) {}
        });

        return v;
    }

}
```
但此时运行，仍然看不到fragment，还需要**将fragment的视图放置到FrameLayout容器中，以添加给activity**。

### 添加 UI fragment 到 FragmentManager

FragmentManager类负责管理fragment并将它们的视图添加到activity的视图层级结构中。fragment事务被用来添加、移除、附加、分离或替换fragment队列中的fragment。这是使用fragment在运行时组装和重新组装用户界面的核心方式。FragmentManager管理着fragment事务的回退栈。

```Java
import android.support.v4.app.Fragment

public class CrimeActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_crime);

        FragmentManager fm = getSupportFragmentManager();

        // 使用R.id.fragmentContainer的容器视图资源ID，向FragmentManager请求获取fragment。如要获取的fragment在队列中已经存在，FragmentManager随即会将之返还。
        Fragment fragment = fm.findFragmentById(R.id.fragmentContainer);
	
        // 如指定容器视图资源ID的fragment不存在，则fragment变量为空值。这时应创建一个新的CrimeFragment，并开启一个新的fragment事务(transaction)，然后在事务里将新建的fragment添加到队列中。
        if (fragment == null) {
            fragment = new CrimeFragment();
            // 创建一个新的fragment事务，加入一个添加操作，然后提交该事物。
            fm.beginTransaction()
                .add(R.id.fragmentContainer, fragment)
                .commit();
        }
    }
}
```

### Fragment生命周期

FragmentManager保持fragment与activity的状态一致，但fragment方法究竟是在activity方法之前还是之后调用的这一点是无法保证的。

---

整理笔记时参考了如下资料：

- 《Android编程权威指南》Bill Phillips  Brian Hardy著，王明发 译。人民邮电第1版。
    英文版书名《Android Programming - The Big Nerd Ranch Guide》

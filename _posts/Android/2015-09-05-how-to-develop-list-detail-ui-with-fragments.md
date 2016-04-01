---
layout: post
title: 如何使用Android UI Fragment开发“列表-详情”界面？
category: Android
tags: [android-fragments, android-viewpager]
---

在移动App里，有几种常见的界面形式：

- 手机上：一个列表界面A，点击某个条目后进入详情界面B，左右滑动可以切换到上/下条的详情界面；
- 平板上：由于屏幕足够大，列表界面A和详情界面B可以同时显示在屏幕上，分列两侧；
- 顶/底部若干标签，点击或者左右滑动可以显示不同的界面。

**在Android上的解决方案之一是`ViewPager + FragmentPagerAdapter + Fragment`，在iOS上的解决方案之一是`UICollectionView + UICollectionViewCell`.**

**《安卓权威编程指南 Android Programming - The Big Nerd Ranch Guide》**这本书的第7~12章、16~22章完成了一个功能复杂的APP开发，对于学习**M(mode)V(view)C(controller)模式**开发非常有益处，并且涉及非常多的主题：

- 如何创建并添加一个Fragment到Activity；
- xml：样式style、主题theme、dp/sp、布局参数layout parameter、边距和内边距margin,padding；
- 使用ListFragment显示列表（介绍了如何使用单例构建数据模型，如何抽象类、ListView是如何从ArrayAdapter获取数据并呈现视图）；
- 如何创建ArrayAdapter来管理ListView的数据；
- 如何响应ListView条目的点击事件；
- 如何定制化ListView条目的布局（默认的布局仅是一个textview）；
- 如何从Fragment中启动并把参数传递给另一个Activity；
- 如何从Activity传递参数给它托管的Fragment（直接获取Activity extra，通过fragment argument bundle）；
- 如何通知Fragment的Hosting Activity返回结果；
- 如何在Fragment内获取返回结果；
- 如何通过ViewPager托管Fragment，以实现屏幕滑动的效果；
- 如何在同一个Activity托管的两个Fragment之间传递数据；
- 如何根据屏幕大小选择布局；

**贰跟着这些章节顺序完整地实现了这个App，对于理解上述主题，非常有帮助。并且通过GitHub记录了实现的过程：[Learning_Android_Criminal_Intent](https://github.com/li2/Learning_Android_Programming/tree/master/Criminal_Intent)**
下面gif展示的内容涉及到Fragment和ViewPager：
![criminal intent app demo](/assets/img/android/CriminalIntent_demo_gif.gif)

**这篇笔记主要是整理总结与Fragment相关的部分，整理自上述提到的章节。**

<!-- more -->

------

## Fragment是什么 & Activity如何管理一个Fragment

> UI Fragment可以管理界面，整屏或部分。有自己的布局文件，包含了用户可以交互的可视化UI元素。
> 用UI Fragment将应用的UI分解成块，利用一个个构建块，很容易做到构建分页界面、动画侧边栏界面等更多其他定制界面。
> Fragment不具有在屏幕上显示视图的能力。因此，只有将它的视图放置在activity的视图层级结构中（称之为**托管Hosting UI Fragment**），fragment视图才能显示在屏幕上。
>
> 在activity代码中添加fragment, 可以在运行时控制fragment,我们可以决定何时将fragment添加到activity中以及随后可以完成何种具体任务；也可以移除fragment，用其他fragment代替当前fragment，然后再重新添加已移除的fragment。
>
> 需要在activity视图层级结构中为fragment视图安排位置，**创建fragment容器布局**：

### step1/3 创建Fragment容器布局

```xml
activity_crime.xml
<FragmeLayout xmlns:android:"https://schemas.android.com/apk/res/android"
  android:id="@+id/fragmentContainer"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  />
```

### step2/3 创建UI Fragment

> 但此时，activity还未托管fragment，所以代码运行后UI看不到任何内容。接下来需要编写代码，**创建UI Fragment**，覆写fragment的生命周期函数（几乎对应到activity的声明周期函数）。
> 创建fragment和创建activity步骤相同：定义布局文件、创建fragment子类、在代码中关联布局文件声明的组件。

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

### step3/3 添加UI Fragment到FragmentManager

> 但此时运行，仍然看不到fragment，还需要**将fragment的视图放置到FrameLayout容器中，以添加给activity**。所以并**没有“start fragment”这个概念**。
> FragmentManager类负责管理fragment并将它们的视图添加到activity的视图层级结构中。fragment transactions（事务）被用来添加、移除、附加、分离或替换fragment队列中的fragment。这是使用fragment在运行时组装和重新组装用户界面的核心方式。FragmentManager管理着fragment transactions的回退栈。

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

        // 如指定容器视图资源ID的fragment不存在，则fragment变量为空值。
        // 这时应创建一个新的CrimeFragment，并创建一个新的fragment transaction用来把新建的fragment添加到队列中。
        if (fragment == null) {
            fragment = new CrimeFragment();
            fm.beginTransaction()
                .add(R.id.fragmentContainer, fragment)
                .commit();
        }
    }
}  
```

> FragmentManager保持fragment与activity的状态一致，但fragment方法究竟是在activity方法之前还是之后调用的这一点是无法保证的。

------

## 使用ListFragment显示列表

> 1. ListView只有在需要显示某些列表项（list item是listview的一个child view object，可以是简单地 view，可以是复杂的view）时，它才会去申请可用的视图对象；如果为所有的列表项数据创建视图对象，会浪费内存；
> 2. ListView找谁去申请视图对象呢？ 答案是adapter。adapter是一个控制器对象，负责从模型层获取数据，创建并填充必要的视图对象，将准备好的视图对象返回给ListView；
> 3. 首先，通过调用adapter的getCount()方法，ListView询问数组列表中包含多少个对象（为避免出现数组越界的错误)；紧接着ListView就调用adapter的getView(int, View, ViewGroup)方法。

### 创建默认的列表项

```java
public class CrimeListFragment extends ListFragment{
    private static final String TAG = "CrimeListFragment";
    private ArrayList<Crime> mCrimes;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        getActivity().setTitle(R.string.crimes_title);
        mCrimes = CrimeLab.get(getActivity()).getCrimes();
        
        // android.R.layout.simple_list_item_1）是Android SDK提供的列表项的布局资源，仅包含一个TextView;
        // 默认的ArrayAdapter<T>.getView(...)方法依赖于toString()方法。
        // toString()方法等价于getClass().getName() + '@' + Integer.toHexString(hashCode()) ，返回了混和对象类名和内存地址的字符串信息。
        ArrayAdapter<Crime> adapter =
                new ArrayAdapter<Crime>(getActivity(), android.R.layout.simple_list_item_1, mCrimes);
        setListAdapter(adapter);
    }
    
    @Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        Crime c = (Crime) l.getAdapter().getItem(position);
        Log.d(TAG, c.getTitle() + " was clicked");
    }
}
```

### 定制列表项

> 1. 创建定义列表项视图的XML布局文件，替代默认的列表项布局文件；
> 2. 创建ArrayAdapter<T>的子类，并override `public View getView(int position, View convertView, ViewGroup parent)`
Get a View that displays the data at the specified position in the data set. 而默认的getView方法调用的是toString方法；
> 3. getView执行时，首先检查是否有复用对象converView，没有则从定制的布局文件中inflate一个新的视图；然后调用Adapter.getItem()方法获取position位置的对象；再然后引用视图对象中的组件，赋予对象的信息；
> 4. 然后在CrimeListFragment中绑定定制的adapter，更新onCreate(...)和onListItemClick(...)
> 5. 由于CheckBox默认是focusable的，点击列表项时会被解读为切换checkbox状态，然后就无法触发onListItemClick()，所以出现在列表项布局内的任何可聚焦组件（如CheckBox或Button）都应设置为非聚焦状态，从而保证用户在点击列表项后能够获得预期效果。

```java
 public class CrimeListFragment extends ListFragment{
         
-        ArrayAdapter<Crime> adapter =
-                new ArrayAdapter<Crime>(getActivity(), android.R.layout.simple_list_item_1, mCrimes);
+        CrimeAdapter adapter = new CrimeAdapter(mCrimes);
     
+    private class CrimeAdapter extends ArrayAdapter<Crime> {
+        public CrimeAdapter(ArrayList<Crime> crimes) {
+            super(getActivity(), 0, crimes);
+        }
+
+        @Override
+        public View getView(int position, View convertView, ViewGroup parent) {
+            // If we weren't given a view, inflate one
+            if (convertView == null) {
+                convertView = getActivity().getLayoutInflater().inflate(R.layout.list_item_crime, null);
+            }
+            
+            // Configure the view for this crime
+            Crime c = getItem(position);
+            TextView titleTextView = (TextView) convertView.findViewById(R.id.crime_list_item_titleTextView);
+            ……
+            return convertView;
+        }
+    }
 }
```

### 重新加载显示列表项
> 如模型层保存的数据发生改变（或可能发生改变），应通知列表视图的adapter，以便其及时获取最新数据并重新加载显示列表项。在适当的时点，与系统的ActivityManager回退栈协同运作，可以完成列表项的刷新。
> CrimeListActivity恢复运行状态后，操作系统会向它发出调用onResume()生命周期方法的指令。CrimeListActivity接到指令后，它的FragmentManager会调用当前被activity托管的fragment的onResume()方法。
> 一般来说，要保证fragment视图得到刷新，在onResume()方法内更新代码是最安全的选择。(因为可能只是暂停，而不是停止onStart方法不会被调用到)

```java
 public class CrimeListFragment extends ListFragment{
+    @Override
+    public void onResume() {
+        super.onResume();
+        ((CrimeAdapter)getListAdapter()).notifyDataSetChanged();
+    }
```

------

## Fragment和Activity之间数据传递

典型的应用场景：ActivityA及其托管的FragmentA，ActivityB及其托管的FragmentB。现在需要从FragmentA中启动并传递数据给ActivityB，ActivityB再把数据传递给FragmentB。

### 从Fragment启动Activity
> 1. 从fragment中启动activity的实现方式，基本等同于从activity中启动另一activity的实现方式。调用Fragment.startActivity(Intent)方法，该方法在后台会调用对应的Activity方法；
> 2. 附加extra信息；
> 3. fragment如何从托管它的activity获取extra信息？简单的方法是getActivity().getIntent()，但牺牲了fragment的封装性，因为它总是需要由某个具体activity托管着。

```java
weiyiWorkCell:Learning-Android-CriminalIntent weiyi$ git diff a31d00d  9d3ebdb
 public class CrimeListFragment extends ListFragment{
     @Override
     public void onListItemClick(ListView l, View v, int position, long id) {
         Crime c = ((CrimeAdapter) l.getAdapter()).getItem(position);
-        Log.d(TAG, c.getTitle() + " was clicked");
+        // Start Activity
+        Intent i = new Intent(getActivity(), CrimeActivity.class);
+        i.putExtra(CrimeFragment.EXTRA_CRIME_ID, c.getId());
+        startActivity(i);
     }

diff --git a/src/me/li2/android/criminalintent/CrimeFragment.java b/src/me/li2/android/criminalintent/CrimeFragment.java
 public class CrimeFragment extends Fragment {
+    public static final String EXTRA_CRIME_ID = "me.li2.android.criminalintent.crime_id";
     // Configure the fragment instance.
     public void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
-        mCrime = new Crime();
+        UUID crimeId = (UUID) getActivity().getIntent().getSerializableExtra(EXTRA_CRIME_ID);
+        mCrime = CrimeLab.get(getActivity()).getCrime(crimeId);
     }
```

### 附加arguments给Fragment，获取arguments
> 1. 每个fragment实例都可附带一个Bundle对象。该bundle包含有key-value对，我们可以如同附加extra到Activity的intent中那样使用它们。一个key-value对即一个argument.
> 2. 首先需创建Bundle对象；然后使用Bundle限定类型的“put”方法（类似于Intent的方法），将argument添加到bundle中。
> 3. **附加argument bundle给fragment，必须在fragment创建后、添加给activity前完成**。所以习惯做法是添加名为newInstance()的静态方法给Fragment类。
> 使用该方法，完成fragment实例及bundle对象的创建，然后将argument放入bundle中，最后再附加给fragment。
> 托管activity需要fragment实例时，需调用newInstance()方法，而非直接调用其构造方法。
> 4. 托管activity就应该知道有关托管fragment方法的细节，但fragment则不必知道其托管activity的细节问题。至少在需要保持fragment通用独立性的时候是如此。

```java
weiyiWorkCell:Learning-Android-CriminalIntent weiyi$ git diff 9d3ebdb acc17af
diff --git a/src/me/li2/android/criminalintent/CrimeActivity.java b/src/me/li2/android/criminalintent/CrimeActivity.java
 public class CrimeActivity extends SingleFragmentActivity { 
     @Override
     protected Fragment createFragment() {
-        return new CrimeFragment();
+        UUID crimeId = (UUID) getIntent().getSerializableExtra(CrimeFragment.EXTRA_CRIME_ID);
+        return new CrimeFragment().newInstance(crimeId);
     }
 }

 public class CrimeFragment extends Fragment {
+    public static CrimeFragment newInstance(UUID crimeId) {
+        Bundle args = new Bundle();
+        args.putSerializable(EXTRA_CRIME_ID, crimeId);
+        
+        CrimeFragment fragment = new CrimeFragment();
+        fragment.setArguments(args);
+        return fragment;
+    }
+    
     @Override
     // Configure the fragment instance.
     public void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
-        UUID crimeId = (UUID) getActivity().getIntent().getSerializableExtra(EXTRA_CRIME_ID);
+        UUID crimeId = (UUID) getArguments().getSerializable(EXTRA_CRIME_ID);
     }
```

### 在Fragment内获取返回结果
> fragment有自己的`Fragment.startActivityForResult(Intent,int)`和`onActivityResult(...)`，但却不具有`setResult(...)`方法，但可以先取得托管它的activity，然后再返回数据。

```java
weiyiWorkCell:Learning-Android-CriminalIntent weiyi$ git diff 8d87138 155c5d8
diff --git a/src/me/li2/android/criminalintent/CrimeFragment.java b/src/me/li2/android/criminalintent/CrimeFragment.java
 public class CrimeFragment extends Fragment {
+    public void returnResult() {
+        getActivity().setResult(Activity.RESULT_OK, null);
+    }
 }

diff --git a/src/me/li2/android/criminalintent/CrimeListFragment.java b/src/me/li2/android/criminalintent/CrimeListFragment.java
 public class CrimeListFragment extends ListFragment{
+    private static final int REQUEST_CRIME = 1;

-        startActivity(i);
+        startActivityForResult(i, REQUEST_CRIME);

+    @Override
+    public void onActivityResult(int requestCode, int resultCode, Intent data) {
+        if (requestCode == REQUEST_CRIME) {
+            // Handle result
+        }
     }
```

------

## ViewPager和Fragment

> 1. 创建CrimePagerActivity类来管理ViewPager；
> 2. 定义包含ViewPager的视图层级结构：以代码的方式构建ViewPager：创建资源id，创建实例，设置实例的id，设置为activity的内容视图content view。
> 3. **在CrimePagerActivity类中关联使用ViewPager及其PagerAdapter（包括FragmentStatePagerAdapter和FragmentPagerAdapter），二者间的配合支持实际归结为两个简单方法的使用，即getCount()和getItem(int)。调用getItem(int)方法获取crime数组指定位置的Crime时，它会返回一个已配置的用于显示指定位置crime信息的CrimeFragment。**
> 4. 修改CrimeListFragment.onListItemClick(...)方法，启动CrimePagerActivity，而非CrimeActivity.
> 5. ViewPager.setCurrentItem()方法设置当前显示的page；
> 6. 使用OnPageChangeListener监听ViewPager当前显示页面的状态变化。onPageScrolled(...)方法可告知我们页面将会滑向哪里；onPageScrollStateChanged(...)方法可告知我们当前页面所处的行为状态，如正在被用户滑动、页面滑动入位到完全静止以及页面切换完成后的闲置状态。

```java
public class CrimePagerActivity  extends FragmentActivity {
    private ViewPager mViewPager;
    private ArrayList<Crime>mCrimes;
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mViewPager = new ViewPager(this);
        mViewPager.setId(R.id.viewPager);
        setContentView(mViewPager);
        
        mCrimes = CrimeLab.get(this).getCrimes();
        
        FragmentManager fm = getSupportFragmentManager();
        mViewPager.setAdapter(new FragmentStatePagerAdapter(fm) {
            @Override
            public int getCount() {
                return mCrimes.size();
            }
            
            @Override
            // 因为需要返回Fragment（用于构建activity），所以在构建adapter时，还需传入FragmentManager给它的构造方法。
            public Fragment getItem(int pos) {
                Crime crime = mCrimes.get(pos);
                return CrimeFragment.newInstance(crime.getId());
            }
        });
        
        mViewPager.setOnPageChangeListener(new OnPageChangeListener() {
            @Override
            public void onPageSelected(int pos) {
                Crime crime = mCrimes.get(pos);
                if (crime.getTitle() != null) {
                    setTitle(crime.getTitle());
                }
            }
            
            @Override
            public void onPageScrolled(int pos, float posOffset, int posOffsetPixels) { }
            
            @Override
            public void onPageScrollStateChanged(int state) {}
        });
        
        UUID crimeId = (UUID) getIntent().getSerializableExtra(CrimeFragment.EXTRA_CRIME_ID);
        for (int i=0; i<mCrimes.size(); i++) {
            if (mCrimes.get(i).getId().equals(crimeId)) {
                mViewPager.setCurrentItem(i);
                break;
            }
        }
    }
}
```

------

## 创建并显示日期选择的对话框（把AlertDialog视图封装在DialogFragment实例中）
 
> 1. 不使用DialogFragment，也可显示AlertDialog视图，但Android开发原则不推荐这种做法。使用FragmentManager管理对话框，可使用更多配置选项来显示对话框，比如：gravity, match_parent.
> 2. 要将DialogFragment添加给FragmentManager管理并放置到屏幕上，可调用DialogFragment.show(FragmentManager fm, String tag)方法。

res/layout/dialog_date.xml

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >
        
        <DatePicker
        android:id="@+id/dialog_date_datePicker"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:calendarViewShown="false"
        />
        
</LinearLayout>
```
src/me/li2/android/criminalintent/DatePickerFragment.java

```java
public class DatePickerFragment extends DialogFragment {
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        // 虽然DatePicker可以直接生成视图，但使用布局文件容易修改对话框的显示内容
        // DatePicker v = new DatePicker(getActivity());
        View v = getActivity().getLayoutInflater().inflate(R.layout.dialog_date, null);

        return new AlertDialog.Builder(getActivity())
            .setView(v)
            .setTitle(R.string.date_picker_title)
            .setPositiveButton(android.R.string.ok, null)
            .create();
    }
}
```
src/me/li2/android/criminalintent/CrimeFragment.java

```java
public static CrimeFragment newInstance(UUID crimeId) {
        mDateButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                FragmentManager fm = getActivity().getSupportFragmentManager();
                DatePickerFragment dialog = new DatePickerFragment();
                dialog.show(fm, DIALOG_DATE);
            }
        });
```

### 同一个Activity托管的两个Fragment如何发送数据
到现在为止，CrimeActivity已经包含了2个Fragment：CrimeFragment（继承自Fragment）和DatePickerFragment（继承自DialogFragment）。
问题是，当用户点击CrimeFragment的日期按钮时，怎么把日期传递给DatePickerFragment？
> 类似从activity传递数据到fragment，替代fragment的构造方法，创建和设置fragment argument通常是在一个newInstance()方法中完成。

```java
weiyiWorkCell:Learning-Android-CriminalIntent weiyi$ git diff 62a68ee 0c6fe64
diff --git a/src/me/li2/android/criminalintent/CrimeFragment.java b/src/me/li2/android/criminalintent/CrimeFragment.java
 public class CrimeFragment extends Fragment {
             @Override
             public void onClick(View v) {
                 FragmentManager fm = getActivity().getSupportFragmentManager();
-                DatePickerFragment dialog = new DatePickerFragment();
+                DatePickerFragment dialog = DatePickerFragment.newInstance(mCrime.getDate());
                 dialog.show(fm, DIALOG_DATE);
             }
         });

diff --git a/src/me/li2/android/criminalintent/DatePickerFragment.java b/src/me/li2/android/criminalintent/DatePickerFragment.java
 public class DatePickerFragment extends DialogFragment {
+    public static final String EXTRA_DATE = "me.li2.android.criminalintent.date";
+    
+    private Date mDate;
+    
+    public static DatePickerFragment newInstance(Date date) {
+        Bundle args = new Bundle();
+        args.putSerializable(EXTRA_DATE, date);
+        
+        DatePickerFragment fragment = new DatePickerFragment();
+        fragment.setArguments(args);
+        
+        return fragment;
+    }
+    
     @Override
     public Dialog onCreateDialog(Bundle savedInstanceState) {
+        mDate = (Date) getArguments().getSerializable(EXTRA_DATE);
         ……
         // 虽然DatePicker可以直接生成视图，但使用布局文件容易修改对话框的显示内容
         // DatePicker v = new DatePicker(getActivity());
         View v = getActivity().getLayoutInflater().inflate(R.layout.dialog_date, null);
         return new AlertDialog.Builder(getActivity())
             .setView(v)
             .setTitle(R.string.date_picker_title)
```

### 同一个Activity托管的两个Fragment如何回传结果

现在用户选择完了日期，点击确定按钮后，如何把这个日期数据从DatePickerFragment回传到CrimeFragment呢？
> Fragment提供来了另外一种绑定方式：为fragment设置数据返回的目标fragment和请求码：调用Fragment.setTargetFragment(Fragment fragment, int requestCode)。
> 然后通过getTargetFragment().onActivityResult(getTargetRequestCode(), int resultCode, Intent data)方法实现数据的回传。

```java
weiyiWorkCell:Learning-Android-CriminalIntent weiyi$ git diff 0c6fe64 5a572dc
diff --git a/src/me/li2/android/criminalintent/CrimeFragment.java b/src/me/li2/android/criminalintent/CrimeFragment.java
 public class CrimeFragment extends Fragment {
+    private static final int REQUEST_DATE = 0;

         mDateButton.setOnClickListener(new View.OnClickListener() {
             @Override
             public void onClick(View v) {
                 FragmentManager fm = getActivity().getSupportFragmentManager();
                 DatePickerFragment dialog = DatePickerFragment.newInstance(mCrime.getDate());
+                dialog.setTargetFragment(CrimeFragment.this, REQUEST_DATE);
                 dialog.show(fm, DIALOG_DATE);
             }
         });

+    @Override
+    public void onActivityResult(int requestCode, int resultCode, Intent data) {
+        if (resultCode != Activity.RESULT_OK) {
+            return;
+        }
+        if (requestCode == REQUEST_DATE) {
+            Date date = (Date) data.getSerializableExtra(DatePickerFragment.EXTRA_DATE);
+        }
+    }
 }

diff --git a/src/me/li2/android/criminalintent/DatePickerFragment.java b/src/me/li2/android/criminalintent/DatePickerFragment.java
 public class DatePickerFragment extends DialogFragment {     
+    private void sendResult(int resultCode) {
+        if (getTargetFragment() == null) {
+            return;
+        }
+        
+        Intent i = new Intent();
+        i.putExtra(EXTRA_DATE, mDate);
+        getTargetFragment().onActivityResult(getTargetRequestCode(), resultCode, i);
+    }
+    
     @Override
     public Dialog onCreateDialog(Bundle savedInstanceState) {
         return new AlertDialog.Builder(getActivity())
             .setView(v)
             .setTitle(R.string.date_picker_title)
-            .setPositiveButton(android.R.string.ok, null)
+            .setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {
+                @Override
+                public void onClick(DialogInterface dialog, int which) {
+                    sendResult(Activity.RESULT_OK);
+                }
+            })
             .create();
     }
+
 }
```

------

整理自《安卓权威编程指南 Android Programming - The Big Nerd Ranch Guide》第7~12章、16~22章。
[你可以从这里获取源码 Learning_Android_Criminal_Intent](https://github.com/li2/Learning_Android_Programming/tree/master/Criminal_Intent)

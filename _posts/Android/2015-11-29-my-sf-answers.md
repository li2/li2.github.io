---
layout: post
title: 分类整理我在 SegmentFault 上针对某些问题作的回答
category: Android
tags: []
---

# Android


## 资源Resource与布局Layout

------

### [android:怎么实现一个控件与另一个指定控件左对齐](http://segmentfault.com/q/1010000003905460/a-1020000003905965)

针对你这种情况，最简单的一种办法是，设置两个TextView的宽度为固定值，且相等。
`LinearLayout`是一种线性排列的布局，布局中的控件从左到右（或者是从上到下）依次排列。`wrap_content`依据其内容分配宽度，“用户名”和“密码”由于内容长度不同，导致对应的两个TextView不等宽，而其后紧跟EditText，这就导致了不对齐。
第2种方法，把LinearLayout中的组件宽度设置为0dp，然后设置其`android:layout_weight`，比如TextView设置为0.2，EditText设置为0.8，这两个控件就会按比例分配整个LinearLayout的宽度。第二个LinearLayout也同样设置，就可以保证EditText对齐。
第3种方法，使用`RelativeLayout`，通过`android:layout_alignLeft="@id/anotherViewId"`设置该View的左边和指定View的左边对齐。


### [Android 布局中 FrameLayout 的作用和 android:layout_weight 属性的工作原理](http://segmentfault.com/q/1010000004040890/a-1020000004041110)

通过代码动态添加的Fragment，需要在布局文件中为 Fragment 添加一个FrameLayout容器，以安排 Fragment 在 activity 视图中的位置。
在FrameLayout视图的子组件中，`layout_weight`属性无效。需要通过 `layout_gravity` 属性值决定子组件在 FrameLayout 视图中的位置。
对于水平方向的 LinearLayout，查看 layou_width 和 layout_weight 以决定子组件的宽度。


### [如何通过代码设置TextView的Margin参数？](http://segmentfault.com/q/1010000004032766/a-1020000004034499)

如果父视图是LinearLayout，那么就可以直接调用textView.setLayoutParams(params)，然后在添加textView到LinearLayout。
如果父视图是RelativeLayout 或者 FrameLayout，上面的做法无效，解决的办法是新建一个LinearLayout，然后把textView添加给它，再把这个LinearLayout添加给父视图。


### [ListView根据KeyBoard弹出、隐藏而自动滚动](http://segmentfault.com/q/1010000004013837/a-1020000004014348)

除了配置当前的Activity `android:windowSoftInputMode="stateVisible|adjustResize"`外，还需要：
`listView.setTranscriptMode(ListView.TRANSCRIPT_MODE_NORMAL);`


### [Android Studio引入图片时报错：Rendering Problems Couldn't resolve resource @mipmap/img1 Failed to convert @mipmap/img1 into a drawable](http://segmentfault.com/q/1010000003983065/a-1020000003983108)

在 `mipmap-[density]` 路径中放app icon图片，仍然在 `drawable-[density]` 中放其它图片。
因此，在Manifest中引用app icon时 `android:icon="@mipmap/ic_launcher"`，在其它需要引用图片的地方仍然用 `@drawable/img1`.


### [如何解决Android Resources$NotFoundException: Resource ID？](http://segmentfault.com/q/1010000003857938/a-1020000003975175)

一定是在调用类似如下API的地方，应该赋予的是资源ID，而你却直接给了一个整数，错的`View.setText(21)`，对的`View.setText(R.string.search);`

### [android ImageView 悬浮文字是怎么实现？](http://segmentfault.com/q/1010000003887701/a-1020000003887739)

实现方式应该是`FrameLayout`, 包含一个占满全部layout的ImageView，一个底部的TextView，一个右上角的用于显示数字的某种view。


<!-- more -->


## Activity

------

### [如何获取一个Android App APK的所有Activity类名，并通过代码启动其中某个Activity？](http://segmentfault.com/q/1010000003878021/a-1020000003899104)

通过一个App: Development.apk 获取一个Android App APK的所有Activity类名。它是默认安装在Android模拟器里的app，目的是方便测试和debug，其中一个功能叫`Package Browser`，可以看到App内的所有的activity.


### [通过startActivity启动第三方应用的Activity时崩溃Permission Denial（比如打开微信朋友圈）](http://segmentfault.com/q/1010000003749035/a-1020000003898581)

只有当Activity向外声明了自己是可以处理某些 intent action 时，第三方 app 才能通过 intent 去启动它，否则将导致崩溃。通过配置文件中的intent过滤器来声明。
当知道了package name和class name，可以这样去启动第三方的activity:
`intent.setComponent(new ComponentName(pkg, cls))`


### [调用系统相机后如何把照片直接传递给一个新的activity？](http://segmentfault.com/q/1010000003888865/a-1020000003889167)

这应该是不可能的，除非你有系统相机的源码，修改源码，然后在系统相机内直接启动你想启动的activity。可是为什么想要这样**直接传递到新的activity**呢？这多么费力不讨好啊。
不论是通过显示的还是隐式的intent，启动第三方的activity，应该总要返回到自己的activity，在`onActivityResult()`中处理返回的数据和结果。
所以我猜，你想实现的那种“直接”的效果，应该是在`onActivityResult`中拿到图片后，再次启动了另一个新的activity。
我做了一个测试：在A中启动B；B返回后，在A的onActivityResult中启动C。**视觉上，B直接跳到了C。**


### [安卓切屏重新调用activity生命周期方法有无必要？](http://segmentfault.com/q/1010000003807732/a-1020000003984641)

在manifest中配置activity的属性 `android:configChanges="keyboardHidden|orientation|screenSize"` 后，就允许Activity自己处理屏幕方向的变化，避免了被系统销毁。也就是说，相应的生命周期方法不会被调到。


### [点击ActionBar的放大镜图标以启动一个包含SearchView的Activity](http://segmentfault.com/q/1010000002539102/a-1020000003970444)

1. 首先配置这个新的Activity为**可搜索的Activity**，并实现其相应的方法；参考 [链接1](https://github.com/li2/Learning_Android_Programming/commit/9f9a899e93efc35cd5190e1f6c9f2f77a6bf6a19)；[链接2](https://github.com/li2/Learning_Android_Programming/commit/9f9a899e93efc35cd5190e1f6c9f2f77a6bf6a19)。
2. 然后为某一个Activity添加MenuItem，在Item的点击事件中启动上面配置的activity；
3. 同时要注意，需要调用`searchView.setIconifiedByDefault(false)`，这样启动的这个可搜索的activity的searchView才是展开的。

### [为什么SearchView提交搜索后onNewIntent()没被调用到？](http://segmentfault.com/q/1010000003871478/a-1020000003885497)

根据API说明，你的问题的关键在于`onQueryTextSubmit()`的返回值：只有返回false，才能使SearchView发起一个intent。如果返回true，就认为这个submit已经被listener自己处理掉了。修改返回值为fasle，解决问题。


### [如何更改ActionBar上的SearchView图标？](http://segmentfault.com/q/1010000003969510/a-1020000003969689)

> Unfortunately, there is no easy way to change the SearchView icon to a custom drawable, since the theme attribute searchViewSearchIcon is not public. See this answer for details.


### [如何实现Android透明导航栏（Translucent Navigation Bar）？](http://segmentfault.com/q/1010000003967799/a-1020000003967988)


### [如何调整Android ActionBar向上按钮(Up button)与屏幕左侧的距离？](http://segmentfault.com/q/1010000003890699/a-1020000003891174)

为action bar提供了自定义的布局，放弃使用android提供的Home键`android.R.id.home`作为UpIndicator。而是在这个布局中添加一个ImageView，这样就可以自定义你需要的间距离。然后实现`ImageView.OnClickListener`，点击时返回父activity。



## Fragment与ViewPager

------

### [Fragment切换时重叠](http://segmentfault.com/q/1010000003947967/a-1020000003948441)

我目前就测试到这两种情况会导致重叠：
可能1.通过代码或者布局文件，向同一个位置添加了2次Fragment；


### [当Android旋转屏幕导致横竖屏切换时，如何保存当前Fragment的实例并在Activity销毁重建后还原状态？](http://segmentfault.com/q/1010000003978225/a-1020000003978395)

- 设备旋转时保存Activity的交互状态: `onSaveInstanceState()`;
- 设备旋转时保存Fragment的交互状态: `setRetainInstance(true)`;
- 设备旋转时保存WebView的数据: `android:configChanges="keyboardHidden|orientation|screenSize"`;
- 设备旋转时保存在自定义View中绘制的图形。


### [(Fragment) page.getCls().newInstance()是什么意思？](http://segmentfault.com/q/1010000003967198/a-1020000003967414)

SimpleBackPage是enum类型，意图是**通过数字获取对应的Fragment类**：

```java
SimpleBackPage page = SimpleBackPage.getPageValue(pageValue);
// 如果pageValue=1，getCls返回的就是FeedBackFragment.class
// 如果pageVaule=2，getCls返回的就是AboutFrament.class
page.getCls();

public enum SimpleBackPage {
    FEEDBACK(1, R.string.setting_about, FeedBackFragment.class),
    ABOUT(2, R.string.setting_about, AboutFrament.class);
    
    private SimpleBackPage(int values, int title, Class<?> cls) {
        this.values = values;
        this.title = title;
        this.cls = cls;
    }
    public Class<?> getCls() {
        return cls;
    }    
```
当拿到Fragment类后，`Fragment.class.newInstance()` 通过调用该类的无参数构造器，创建并返回该类的一个实例，等价于：`new Fragment()`。


### [viewpager onPageSelected没有执行](http://segmentfault.com/q/1010000003901495/a-1020000003901719)

`this.viewPager.setOnPageChangeListener(this);`
追问：“首次进入界面时,显示第一个页面时相应的按钮颜色已经变成点击后的状态？”。
追答：首次进入界面时onPageSelected不会被调到。可以通过viewPager.setCurrentItem(0) 触发它。


### [PagerAdapter.notifyDataSetChanged时崩溃：Activity has been destroyed](http://segmentfault.com/q/1010000003880819/a-1020000003880863)

`Fragment`是attach在`Activity`上的，crash的原因显然是你在尝试调用`notifyDataSetChanged`更新fragment时，该fragment的attached activity已经被销毁了：“IllegalStateException: Activity has been destroyed”。一个workaround是这样的：

```java
// here you check the value of getActivity() and break up if needed
if(getActivity() == null) {
    return;
}
// do your stuff to update fragment
// ...
```


### [ViewPager的Fragment中嵌套的Fragment怎么实现刷新数据?](http://segmentfault.com/q/1010000003731705/a-1020000003738254)

我按照你的流程写了一个demo，复现了你描述的bug现象。
当滑动ViewPager时，导致父fragment视图被销毁，即`onDestroyView()`被调到，
再次滑动到该父fragment时，重建视图，即`onCreateView()`被调到，`ft.add()`被再次调到，再次添加2个子fragment，这就导致了你提到的问题，而并非是`ft.hide()`不起作用。
一个变通的方法可以解决这个问题，在commit之前先remove所有的子Fragment：


### [放入ViewPager的Fragment会因为Activity发生重新启动也跟着Activity重新创建吗？](http://segmentfault.com/q/1010000003719990/a-1020000003723522)

你的问题应该从生命周期和ViewPager的adapter来理解。
Fragment具有和Activity相似的生命周期，并且**其生命周期方法由托管它的Activity调用。**
当把一些Fragment放入ViewPager时，就需要adapter的支持，以管理这些Fragment。对于常用的两种adapter，FragmentStatePagerAdapter会销毁掉不需要的fragment，而FragmentPagerAdapter只是销毁了fragment的视图。


### [如何在屏幕底部显示DialogFragment对话框，并且与屏幕等宽？](http://segmentfault.com/q/1010000003883964/a-1020000003884103)

```java
public class DatePickerDialog extends DialogFragment {
  @Override
  public Dialog onCreateDialog(Bundle savedInstanceState) {
    // 使用不带theme的构造器，获得的dialog边框距离屏幕仍有几毫米的缝隙。
    // Dialog dialog = new Dialog(getActivity());
    Dialog dialog = new Dialog(getActivity(), R.style.CustomDatePickerDialog);
    // must be called before set content    
    dialog.requestWindowFeature(Window.FEATURE_NO_TITLE);
    dialog.setContentView(R.layout.dialog_datepicker);
    
    // 设置宽度为屏宽、靠近屏幕底部。
    Window window = dialog.getWindow();
    WindowManager.LayoutParams wlp = window.getAttributes();
    wlp.gravity = Gravity.BOTTOM;
    wlp.width = WindowManager.LayoutParams.MATCH_PARENT;
    window.setAttributes(wlp);
 
    return dialog;
  }
```


## Intent

------

### [PendingIntent和Intent传递数据有区别吗？](http://segmentfault.com/q/1010000002563862/a-1020000003928253)

`PendingIntent.getActivity(...)` 和 `startActivity(intent)` 两种方式启动的Activity都以同样的方法获取extra数据：
`PendingIntent.getActivity()`打包了一个`Context.startActivity()`方法的调用，该方法告诉操作系统『我需要启动一个Activity』，随后调用`PendingIntent.send()`方法时，操作系统会按照我们的要求发送原来封装的intent。
但是使用PendingIntent需要特别注意：

1. 获取的extra数据很可能是旧的：除非把第4个参数从0改成`PendingIntent.FLAG_UPDATE_CURRENT`.
2. 为了安全因素，最好只发送显示的intent.

`PendingIntent.getActivity(...)` 和 `startActivityForResult(...)` 两种方式（第2个参数都是requestCode）启动的Activity以同样的方法返回result：`setResult(resultCode, intent)`。
然后在`void onActivityResult(int requestCode, int resultCode, Intent data)`中根据`resultCode`做相应的处理。

总结：还是让`PendingIntent`做它“字面上”该做的事情为好，比如通过AlarmManager定时做的任务，通过NotificationManager发送消息到通知栏，一种pending状态。让`startActivity`启动应用内的Activity，启动其它应用：读取联系人、调用摄像头、发送社交圈，等等，需要立刻响应的任务。



## ListView与ExpandableListView

------

### [如何删除Android ExpandableListView中某个group item的child item？](http://segmentfault.com/q/1010000003972290/a-1020000003973789)

要正确定义数据，以及初始化adapter的数据。这样定义数据是不可取的：

```java
rivate String [] groupStr={"第一组","第二组","第三组"};
private String [] childStr={"first","second","third"};
private List<Map<String, String>> groupData = new ArrayList<Map<String, String>>();
private List<Map<String, String>> childData = new ArrayList<Map<String, String>>();
```
如果删除了child index=2元素，必然导致你的childData和groupData数量不对等，当adapter试图为group2调用`getChildrenCount(int groupPosition)`时崩溃。
应该像这样定义child data: `List<List<Map<String, ?>>>`.


### [删除ExpandableListView的child item时报错ClassCastException: Activity cannot be cast to OnClickListener](http://segmentfault.com/q/1010000003909527/a-1020000003910783)

原因是你传入的context，也就是`SumFileActivity`，并没有实现你定义的`IOnClickListener`。所以无法强制转型。既然你在Fragment中实现了接口，那我们可以这样做，注册一个接口，替代传入context的办法，然后还需要做的事件就是，在创建MyDialog的Fragment里面调用`setOnClickListener`.


### [ExpandableListView中自定义的child item无法点击](http://segmentfault.com/q/1010000003902645/a-1020000003903355)

虽然设置了`setOnChildClickListener()`，但点击child list view item无反应，就我刚才测试来看，有两种情况：
第1种情况，`isChildSelectable()`返回`false`，已经被你排除了。
第2种情况，item的布局文件中，有控件劫持了点击事件（`ListView`都存在这种情况）`android:focusable="true"`。不配置TextView的这个属性（因为TextView默认是非聚焦的），或者设置为false，就能让item能响应点击事件。
而像CheckBox, Button, EditText等默认是可聚焦的，如果包含在list item layout内，而且还需要响应item的点击事件的话，那么必须设置为非聚焦。


### [ExpandableListAdapter中getChild和getGroup调用时间？](http://segmentfault.com/q/1010000003953634/a-1020000003954043)

关于getChildrenCount：
点击展开GroupItem时，`getChildrenCount()`被调到，以返回这个Group的child数量（为避免出现数组越界的错误）；然后adapter才会去调用`getChildView()`。

关于getChildView()、getChild()、getGroup()：
我们需要覆写`getChildView()`，以填充视图，那么首先要拿到数据，而`getChild()`的目的就是拿到存储在adapter中对应位置的数据。当然，如果你在adapter之外维护了一个child data list，也可以直接从这个list中取数据。但是`getChild()`看起来不是更清楚明了吗？
`getGroup()`也是同样的道理，在执行`getGroupView()`填充group视图时，让你可以轻松地获取对应的数据。

本质上，ExpandableListAdapter 的`getChild(), getGroup()` 和 android.widget.Adapter 的 `Object getItem(int position)` 是一回事儿：『Get the data item associated with the specified position in the data set.』


### [如何理解BaseAdapter.getVeiw()参数convertView的null与非null](http://segmentfault.com/q/1010000003870722/a-1020000003870759)

简单来说，是为了**复用，避免每次从layout资源文件生成新的视图（或者是通过代码生成新的视图）**。比如像`ListView`或者`GridView`，屏幕上若能显N个条目，那么`getView`就被调用N次，以提供对应位置的视图。而复用之前已经生成的视图，可以提高效率。


### [长按Fragment List Item无法弹出上下文菜单context menu](http://segmentfault.com/q/1010000002979042/a-1020000003862755)

你的代码已经成功地添加了context menu, 之所以长按无反应，原因很可能是list item有组件截获了list的点击事件。检查list item的布局文件，应该可以看到`android:focusable="true"`。把它设置为false.


### [使用默认的ArrayAdapter构建ListView时抛出异常NullPointerException](http://segmentfault.com/q/1010000003835627/a-1020000003836000)

错误提示信息告诉你问题出在这：『at android.widget.ArrayAdapter.getView(ArrayAdapter.java:369』
你需要了解默认的getView方法的实现：
在adapter的构造方法中指定的布局(android.R.layout.simple_list_item_1)是Android SDK 提供的预定义布局资源。该布局拥有一个TextView根元素。而默认的ArrayAdapter<T>.getView(...)实现方法依赖于toString()方法。它首先生成布局视图，然后找到指定位置的对象并对其调用toString()方法，最后得到字符串信息并传递给TextView。
所以，我猜问题应该是你传给Adapter的dataList包含null对象。你应该着重检查更改adapter数据的代码。


## Service

------

### [Android后台下载问题](http://segmentfault.com/q/1010000004001588/a-1020000004005912)

通过bound service实现后台下载，这样，显示下载状态的UI组件被销毁再重建时，通过绑定service就可以正确显示下载状态。当然下载任务必须交给工作线程（比如AsyncTask），由bound service启动AsyncTask，通过广播或者接口的方式，应用组件就可以获取下载状态。


### [如何根据数据库是否有新内容来给手机发送一条push?](http://segmentfault.com/q/1010000003937375/a-1020000003937397)

针对Android，如果App不在前台的话，可以通过service完成：

- 使用IntentService在后台抓取服务器的最新数据；
- 使用AlarmManager和PendingIntent安排服务的运行（设置service查询的间隔时间）；
- 使用Notification从后台通知用户（用户下拉通知栏后可以看到详细信息，点击后进入App）。



## Broadcast与Receiver与Notification

------

### [为何Android中大部分锁屏APP都要手动勾选“通知使用权”(Notification Access)？](http://segmentfault.com/q/1010000004029184/a-1020000004034635)

从官方文档看，应该是为了获得其他APP的通知内容从而在自己的锁屏界面上显示。you can designate whether a notification from your app is visible on the lock screen.


### [如何不在状态栏显示notification的ticker view，仅在状态栏的下拉抽屉中显示notification的content view？](http://segmentfault.com/q/1010000004012291/a-1020000004013049)

在4.2的机器上测试，无法做到。


### [application 进程被系统杀死后，为何不能接受broadcast？](http://segmentfault.com/q/1010000003915994/a-1020000003916834)



## Thread

------

### [假如主线程依赖子线程A的执行结果，如何让A执行完成之后主线程再往下执行呢？](http://segmentfault.com/q/1010000003961853/a-1020000003962008)

需要在子线程执行完成的地方，通过主线程的Handler发送一条消息；主线程收到消息后执行。
主线程是UI线程，不要试图让UI线程等待某个结果，之后再往下执行，这会导致UI卡顿。UI线程是一直循环的，我们需要通过消息机制通知UI线程去做一些事情。


### [关于线程上的问题](http://segmentfault.com/q/1010000003030312/a-1020000003952606)

你既然想在主线程之外执行`recorder.start()`，所以**关键在于你代码中的mhandler是谁创建的？** 如果是在UI线程中创建的，那么你通过`mhandler.sendMessage`或者`post(Runnable)`这些方式，相应的代码仍将在主线程中执行。



## View

------

### [如何处理 ScrollView 的点击事件？](http://segmentfault.com/q/1010000003945505/a-1020000003945637)

方法1：ScrollView內的子View消耗了它的点击事件，所以一个解决办法是，为它的子View设置`setOnClickListener`. (这种办法，我测试了可行）
方法2：设置所有子View的xml属性`android:clickable="false"`, 然后实现ScrollView的`setOnClickListener`方法。（这种方法来自[OnClickListener on scrollView](http://stackoverflow.com/a/16776927/2722270)，我测试了不行，但解决了别人的问题）


### [怎么判断一个图片数组里面的id被点击事件？](http://segmentfault.com/q/1010000003944754/a-1020000003944788)

为数组里的每一个图片设置一个tag：`ImageView.setTag(Objcet tag)`，
在onClick中通过tag识别图片：`((ImageView)v).getTag()`


### [SlidingMenu拖拽出来是白板，怎么办？](http://segmentfault.com/q/1010000003937809/a-1020000003938555)

`R.layout.menu_frame`包含一个`FrameLayout`，这只是一个空的布局，你可以把它当做一个Fragment的容器。SlidingMenu提前注册了一个空的布局，需要你用自己的menu去替换它：
`.replace(R.id.menu_frame, new SampleListFragment())`
你并没有替换，所以看到的是空的。这种做法的本质是把一个包含menu的Fragment视图添加给Activity。


### [点击button弹出单选框，选择其中一个值，并显示在button上](http://segmentfault.com/q/1010000003925031/a-1020000003925773)

```java
AlertDialog.Builder builder = new AlertDialog.Builder(this);
builder.setSingleChoiceItems(R.array.test, 0, new OnClickListener() {
    @Override
    public void onClick(DialogInterface dialog, int which) {
        ListView lw = ((AlertDialog) dialog).getListView();
        // which表示点击的条目
        Object checkedItem = lw.getAdapter().getItem(which);
        // 既然你没有cancel或者ok按钮，所以需要在点击item后使dialog消失
        dialog.dismiss();
        // 更新你的view
        mButton.setText((String)checkedItem);
    }
});

AlertDialog dialog = builder.create();
dialog.show();
```

### [TextView 如何设置半透明背景色？](http://segmentfault.com/q/1010000003890144/a-1020000003890302)

`android:alpha="0.5"`
> alpha property of the view, as a value between 0 (completely transparent) and 1 (completely opaque). Related Methods: `setAlpha(float)`


### [对于一个单行TextView，当字符串超出一行时，如何获取未显示的部分字符串？](http://segmentfault.com/q/1010000003794567/a-1020000003889360)

如果你想得到已显示的字符个数，或者未显示的字符个数，那么其中的关键是如何计算每一个字符的宽度。然后遍历这个字符串，当前n个字符宽度总和，超过TextView宽度时，就得到了已显示的字符个数。


### [如何修改WebView文本选中时的Contextual Action Bar为Floating Context Menu？](http://segmentfault.com/q/1010000003881268/a-1020000003881616) 



## 数据存储

------

### [Android开发中sqlite的使用广泛吗？](http://segmentfault.com/q/1010000003958546/a-1020000003959107)

- 如果需要**存储用户配置**，比如某些选项是允许还是禁止，或者是搜索框的历史记录，可以使用 **shared preferences**，它是一种存储key-value的xml文件，可以实现轻量级数据的永久存储，使用SharedPreferences类读写。
- 如果需要**存储少量的、简单的数据**，可以使用**Json**文件。
- 如果需要**存储大量的、复杂的数据**，比如一个跑步运动App，持续追踪用户的跑步路线，那就需要存储大量的地理位置数据。而**SQLite**是一个轻量级的开源跨平台库，并具有一套强大的关系型数据库API可供使用，数据在磁盘上单个文件的形式存在。Android为SQLite提供了很多类，可以很方便的完成对数据库的读写操作。


### [如何理解Android ContentResolver.query(...) 参数为null时的作用？](http://segmentfault.com/q/1010000004023270/a-1020000004035066)

对API存疑的时候，最快的方式是查看API文档：Passing null will return all columns / all rows for the given URI.


### [如何解析Json文件？](http://segmentfault.com/q/1010000003750099/a-1020000003750597)

我们把json文件重新排列，目的是为了呈现其清晰的嵌套结构。因此，可以注意到，json包含这些元素：
由`[]`括起来的称之为数组`JSONArray`;
由`{}`括起来的称之为`JSONObject`;
字符串`String`；
`boolean, double, int, long` 基本数据类型；

通过JSON Object的方法获取这些数据，比如string：`JSONObject.getString(String name)`
然后根据JSON文件的结构，一层一层地解析。



## Java

------

### [什么是回调函数？一个类中的回调函数是什么作用？](http://segmentfault.com/q/1010000003775109/a-1020000003894563)

发现你最近也有提Android相关的问题，那我就通过Android ListView item的点击事件，一种使用频率很高的view，通过它认识“回调”，可能有助于理解。
比如ListView包含几个概略信息条目，你想点击某个条目跳到详情界面。

`mListView.setOnItemClickListener(new OnItemClickListener() { ... });`
就已经实现了一个回调（implements a callback interface）。
接下来发生的事情我们就知道了，点某个条目就跳到其详情界面。
问题是，谁去调(call)它呢？我们在实现(implements)这个回调时，为什么必须要override其中的方法呢？

// 当view被点到时，performItemClick就被调到，处理点击带给view的变化，除此之外，
// AdapterView还想到了，“如果其它类想在点击发生时做点儿事情，该怎么办呢？”
// 通过接口。
// (1)我来定义接口，把知道的信息（被点到的view、position、id）告诉你；
// (2)你来实现接口，然后把实现的接口告诉我。


### [OnItemClickListener是接口有没有被子类继承？](http://segmentfault.com/q/1010000003849057/a-1020000003849179)

OnItemClickListener是接口，不是内部类。接口需要你去implement。
一般的做法是：`listview.setOnItemClickListener(mListener)`，然后定义这个listener，并实现其接口规定的方法。
不理解你如此定义 `new ListView. OnItemClickListener()` 的意图，但之所以出错，也许你是没有导入 `import android.widget.AdapterView.OnItemClickListener`。


## 其它Android相关问题

------

### [保存在Application子类中的全局变量什么情况下会丢失？](http://segmentfault.com/q/1010000004042130/a-1020000004042558)

可以继承 `Application` 类以保存全局的“application state”。因为这些变量是全局的，所以应用的所有组件都可以获取/更改这些变量。多数情况下，静态单例模式可以提供相同的功能。


### [命令行程序如何获取系统（Android）已安装应用信息？](http://segmentfault.com/q/1010000003970708/a-1020000003970819)

```sh
shell:/ $ pm list packages
shell:/ $ pm list packages -f
shell:/ $ dumpsys package com.tencent.mm | grep versionName
```


### [Android Studio中如何开发系统应用？](http://segmentfault.com/q/1010000002907252/a-1020000003952655)
这样的 app 要放在 Android bsp 相应的目录中，通过 Android.mk 文件编译，这才能获取某些特殊权限，这些权限是通过集成编译器无法获取的。



# Linux

------

### [Linux Shell变量替换操作](http://segmentfault.com/q/1010000003992911)

### [在linux上添加crontab任务的问题](http://segmentfault.com/q/1010000003961169/a-1020000003961191)


# iOS

------

### [如何自定义一个UIView？](http://segmentfault.com/q/1010000003967071/a-1020000003967789)

- 需要在Storyboard中把UIView的类名修改为你自定义的类名：点击位于Storyboard的CustomView，在右侧的选项卡中，点击Identity Inspector选项卡，然后修改Custom Class；
- 然后把这个CustomView从Storyboard中拖到ViewController中，建立IBOutlet；
- 这样你就拥有了这个CustomView的实例了，然后就可以修改它的属性，调用它的方法。

可以参考[这个demo工程](https://github.com/li2/Learning_iOS_Programming/tree/master/CustomView)。如果你是想通过Storyboard来完成这件事情，就用不着initWithframe，这个方法的目的是用代码生成一个指定frame的view。


### [自定义cell中的图片的约束应该如何定义？](http://segmentfault.com/q/1010000003965903/a-1020000003966212)


### [如何仅改变iOS某一个控制器导航栏的隐藏或透明度？](http://segmentfault.com/q/1010000003965674/a-1020000003966161)


### [自定义的UICollectionViewCell, for循环创建btn, 用代理设置btn的绑定事件无效怎么回事?](http://segmentfault.com/q/1010000003961338/a-1020000003962248)

```c
@interface MainViewController () <FristSectionCollectionViewCellDelegate>
self.fristSectionCollectionViewCell.delegate = self;
```
虽然你实现了delegate相应的方法，但如果不给delegate赋值，那么如下的判断将为false：
`if ([_delegate respondsToSelector:@selector(choseTerm:)])`。[一个demo](https://github.com/li2/Learning_iOS_Programming/tree/master/CollectionViewButtonDelegateTest)。


### [如何找出不必要的约束constraints in following list is one you don't want](http://segmentfault.com/q/1010000003838767/a-1020000003838864)

严格讲，上述提示并不是出错的约束，而是多余的约束，比如设置了上、下边距，同时又pin了高度，就会导致出现这样的问题。感觉你的问题应该是设置了UIView:0x166ea400高度为37，然后又设置它等于另一个UIView:0x16526700宽度的0.0894。这样就冗余了。
而排查的方法是，在storyboard中挨个找UIView，37是个很好的切入点，我是这样做的，很笨。


### [UICollectionView 如何显示它全部的内容？](http://segmentfault.com/q/1010000003729136/a-1020000003734992)

如果你想实现“不需要拖动就可以显示UICollectionView的全部内容”，前提是你为UICollectionView分配的layout必须要容得下所有的UICollectionViewCell.
如果满足这个前提，比如你想显示的UICollectionView包含9个cell，每个cell大小相同，就像一个九宫格。
你必须设置每个cell的frame size：

```c
// 设置指定位置cell的frame size
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath {
    if (collectionView == self.myCollectionView) {
        CGRect collectionViewFrame = self.myCollectionView.frame;
        CGFloat cellWidth = collectionViewFrame.size.width/kCollectionViewCols;
        CGFloat cellHeight = collectionViewFrame.size.height/kCollectionViewRows;
        return CGSizeMake(cellWidth, cellHeight);
    }
}
```


### [子view根据父view 垂直居中](http://segmentfault.com/q/1010000002647958/a-1020000002647984) 

根据父view的`frame.origin.x`和`frame.size.width`，以及子view的宽度，计算子view的横坐标x，应该是`父x + 父w/2 - 子w/2`，然后`CGRectMake`。



# C

------

### [指针作为函数的参数](http://segmentfault.com/q/1010000003955029/a-1020000003955335)


### [sizeof是如何计算数组大小的？](http://segmentfault.com/q/1010000003848156/a-1020000003848465)

严格讲，你这句话『sizeof(arr) =10; 这里只是把地址传给sizeof啊』是错误的，**你传的是数组名，数组名不等价于地址**。
编译器用数组名标记数组的属性，比如具有确定数量的元素。而你说的**地址**，也就是**指针**，只是一个标量值。
只有当数组名在表达式中使用时，编译器才会为它产生一个指针常量。而只有以下两种情况，才不被当做指针常量：

- sizeof(数组名)：返回数组长度（所占的字节数，不是数组元素个数），而不是指向数组的指针的长度。
- &数组名：产生一个指向数组的指针，而不是一个指向某个指针常量的指针。

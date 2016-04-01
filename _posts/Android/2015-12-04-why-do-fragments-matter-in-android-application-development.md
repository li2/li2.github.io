---
layout: post
title: 「译」向Big Nerd Ranch提问：为什么Fragment在Android App开发中非常重要？
category: Android
tags: [android-fragments, android-viewpager]
---

译者说：

《Android权威编程指南 The Big Nerd Ranch Guide》是一本非常优秀的 android app 开发入门书籍，这本书通过几个或复杂或简单的 app 开发，循序渐进（绝不是填鸭式）地引导初学者学习 androi app 开发的各种知识。而书中几乎所有应用的界面都是基于 fragment 来构建的。

书中的第7~12章、16~22章完成了一个功能复杂的APP开发，可以学习到：

- 如何使用 Fragment 构建『列表-详情』界面；
- 如何使用 ViewPager 滑动屏幕以切换不同的『详情』界面；
- 如何根据屏幕大小选择布局；
- 如何在 fragment 间，以及在 activity 和 fragment 间传递数据；
- 其它知识点：操作栏、上下文菜单、隐式 intent、JSON、定制 ListViewItem、MVC；

书中的第14章专门讲了如何在设备旋转时保存 Fragment 的交互状态。而设备旋转时需要处理的其它事情包括『设备旋转时保存Activity的交互状态』在3.2、3.3；『设备旋转时保存WebView的数据』在31.3；『设备旋转时保存在自定义View中绘制的图形』在32.5。

**而这篇文章几乎可以看作是对上述所有内容的综述，作者试图告诉我们为什 fragments 如此重要**。如果你读过这本书，你可以把这篇文章当作总结；如果你没读过，可以当作概览。

且听他讲。

[原文出处](http://www.informit.com/articles/article.aspx?p=2126865)， Aug 13, 2013，by Big Nerd Ranch.
译者：li21, weiyi.just2@gmail.com, 2015-12-04, 禁止转载

------

# Ask Big Nerd Ranch: Why do Fragments Matter in Android Application Development?

> **Q:** Why does your book Android  Programming heavily advocate the use of fragments when developing Android applications? You can develop Android applications without using any fragments, so why bother? Why do fragments matter?

问：为什么在你的书里极力倡导使用 Fragment 开发 Android App 呢？不使用 Fragment 也可以开发 App，为什么要绕个弯子呢？为什么认为 Fragment 如此重要？

<!-- more -->

> **A:** As Android developers, we have two main controller classes available to us: Activities and Fragments.
>
> Activities have been around for a very long time and are used to construct a single screen of your application. When someone is using your Android application, the views that the user sees and interacts with are hosted and controlled by a single activity. Much of your code lives in these Activity classes, and there will typically be one activity visible on the screen at a time.
>
> As an example, a simple activity may look like the following:

答：作为 Android 开发者，可以使用的两个主要的 controller classes 是 Activity 和 Fragment.

Activities 已经被用于构建单屏界面很长时间了。用户使用你的 App 时所看到的、与之交互的界面，是被某个 activity 托管和控制的。大部分代码存在于这些 Activity 类内，某个时刻通常会有一个 activity 处于可见状态。

举个栗子，一个简单的 activity 看起来像这样：

```java
public class MainActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```
> When Honeycomb was released, the Fragment class was introduced. Fragments are another type of controller class that allows us to separate components of our applications into reusable pieces. Fragments must be hosted by an activity and an activity can host one or more fragments at a time.
>
> A simple fragment looks similar to an activity, but has different life cycle callback methods. The fragment below accomplishes the same thing as the activity example above: it sets up a view.

Fragment 类是在 Honeycomb（*译注：蜂巢 3.0 API level 11*）发布时被介绍的。它允许我们把应用组件分解成可重用的部件。Fragment必须被一个 activity 托管，而一个 activity 可以一次托管一个或多个 fragment。

一个简单的 fragment 看起来和 activity 很像，但是具有不同的生命周期回调方法。下面的 fragment 代码段和上面的 activity 代码段完成了相同的事情：建立视图。

```java
public class MainFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_main, container, false);
    }
}
```
> As you can see, activities and fragments look very similar and are both used to construct applications. So, what are the advantages that come with the use of Fragments?

如你所见，activities 和 fragments 看起来非常像，同样被用于构建app。所以，使用 fragments 可以带来什么优势呢？且听我一一道来：

## Device Optimization 设备优化

> As I mentioned before, fragments were introduced with the Honeycomb release of Android. Honeycomb was the first version of Android with official support for tablets. One of the original goals of the fragment concept was to help developers make applications that provide different user experiences on phones and tablets.
>
> The canonical example of fragment use is an email client. Let’s take a look at the Gmail app. When running on a phone, the Gmail app will display a list of the user’s emails. Tapping on one of these emails will transition the screen to showing the detail view for that particular email. This process is composed of two separate screens.
>
> When using the same app on a tablet, the user will see both the email list and the detail view for a single email on the same screen. Obviously, we can show much more information at once with a tablet.
>
> Fragments make implementing this functionality easy. The list of emails would exist in one fragment, and the detail view for an email would exist in another fragment. Depending on the device size, our activity will decide to show either one or both of these fragments.
>
> The beauty of fragments here is that we do not have to modify either of these Fragment classes for this to happen. Fragments are reusable components of our application that can be presented to the user in any number of ways.

我刚刚讲过，Honeycomb 发布时引入了 fragments。Honeycomb 版本首次对平板设备提供了正式支持。fragment 理念的最初目标之一是帮助开发者在构建手机和平板应用时提供不同的用户体验。
邮件客户端是用到 fragment 的典型例子，比如 Gmail App。手机版展示了用户的邮件列表，点击其中一个就跳转到展示邮件内容的详情界面。这个过程包含了两个界面，各自独占屏幕。
当用户在平板上使用 Gmail App 时，可以在一个屏幕上同时看到列表界面和详情界面。显然我们可以在平板上一次显示更多的信息。
Fragments 很容易实现这个功能。列表界面是一个 fragment，详情界面也是一个 fragment。activity 根据屏幕大小决定显示其中一个或全部。
Fragments 的优雅之处在于，不需要修改 fragment 类的代码就可以完成上述事情。作为 app 的可重用组件，fragments 可以以任意多种方式呈现给用户。

![Figure 1 Fragments in use on a tablet](/assets/img/android/Fragments_in_use_on_a_tablet.png)

> So, this is great, but what if you aren’t developing an app that works on phones and tablets? Should you still use fragments?
>
> The answer is yes. Using fragments now will set you up nicely if you change your mind in the future. There is no significant reason not to start with fragments from the beginning, but refactoring existing code to use fragments can be error-prone and will take time. Fragments can also provide other nice features besides UI composition.

这非常棒，但如果你开发的应用不需要兼容手机和平板，还需要使用 fragments 吗？

答案是『yes』。如果将来你改变了主意*（译注：结合上下文，应该理解为需要大刀阔斧地改代码）*，你会庆幸当初使用了 fragments。从一开始并没有特别的理由不使用 fragments，但是为了使用 fragments 而重构已有的代码，将是耗时且容易出错的。除了构成UI外，Fragments 还提供了一些其它优秀特性。

## ViewPager

> For example, implementing a ViewPager is very easy if you use fragments. If you’re not familiar with ViewPager, it’s a component that allows the user to swipe between views in your application. This component is used in many apps, including the Google Play Store app and the Gmail app. Swiping just swaps out the fragment that the user is seeing.
> 
> ViewPager hits at one of the core ideas behind fragments. Once your application-specific code lives in fragments, those fragments can be hosted in a variety of ways. The Fragment is not concerned with how it’s hosted; it just knows how to display a particular piece of your application. Our activity class that hosts fragments can display one of them, multiple Fragments, or use some component like a ViewPager that allows the user to modify how the fragment is displayed. The core pieces of code in your app do not have to change in order for you to support all of these different display types.

比如，使用 fragments 可以很方便地实现 ViewPager。如果你不熟悉 ViewPager，那我就简单说说，它作为一个应用组件，允许用户滑动屏幕以切换界面。很多 app 用到了它，包括 Google Play Store app 和 Gmail app。滑动屏幕就置换了当前的 fragment。

ViewPager 瞄准了 fragments 的一个核心理念。一旦你的特定的应用代码存在于 fragments，这些 fragments 就可以以多种方式被托管。Fragment 不需关心如何被托管，它只需知道如何展示你的应用的特定部分。托管 fragments 的 activity 类，可以显示一个 fragment；或者多个 fragment；或者像 ViewPager 那样通过某种方式显示 fragments。不需要修改应用的核心代码就可以支持这些不同的显示方式。

## Other Features 其它特性

> So, Fragments can help us optimize our applications for different screen sizes and they help us provide interesting user experiences. There are other features that Fragments can help with as well.
>
> You can think of Fragments as a better-designed version of an activity. The life cycle of a Fragment allows for something that Activities do not: Separation of the creation of the View component from the creation of the controller component itself.

所以 fragments 可以帮助我们优化应用以适应不同的屏幕尺寸，可以帮助我们为用户提供有趣的体验。除此之外，它的其它特性也可以帮到我们。

你可以把 fragment 当作一个『设计更合理』的 activity。Fragments 的生命周期允许它做一些 activity 做不到的事情：它分开了实例的创建和视图的创建。

译注：作者是指 Fragment 具有 `onCreate(...)`, `onCreateView(...)`, `onDestroyView()` 等生命周期方法，这些方法使得 fragment 拥有了分开创建实例和视图、以及保存实例而仅销毁视图的能力。而 Activity 的 `onCreateView(...)` 并不是生命周期回调方法。

> `onCreate(Bundle)` called to do initial creation of the fragment.
> `onCreateView(LayoutInflater, ViewGroup, Bundle)` creates and returns the view hierarchy associated with the fragment.
> `onDestroyView()` allows the fragment to clean up resources associated with its View.

*译注部分结束。*

> This separation allows for the existence of retained fragments. As you know, on Android, rotating your device is considered a configuration change (there are a few other types of configuration changes as well). This configuration change triggers a reload of Activities and Fragments so that resources can be reloaded for that particular configuration. During the reload, these instances are completely destroyed and then recreated automatically. Android developers have to be conscious of this destruction because any instance variables that exist in your activity or fragment will be destroyed as well.
> 
> Retained fragments are not destroyed across rotation. The view component of the fragments will still be destroyed, but all other state remains. So, fragments allow for the reloading of resources across rotation without forcing the complete destruction of the instance. Of course, there are certain situations where this is and is not appropriate, but having the option to retain a fragment can be very useful. As an example, if you downloaded a large piece of data from a web server or fetched data from a database, a retained fragment will allow you to persist that data across rotation.

这种『分离』允许我们保存 fragment 实例。你知道的，旋转 android 设备屏幕被认为是设备配置发生了变化（配置变化还包括其它几种）。系统会寻找更合适的资源以匹配配置，这个过程中 activities 和 fragments 实例会被系统销毁，然后新的实例会被创建。android 开发者必须考虑这种销毁的情形，因为保存在 activity 和 fragment 中的所有实例变量也同样被销毁了。

『Retained fragments』在设备旋转时实例不会被销毁，仅仅是视图被销毁了，其它所有状态都会被保存下来。尽管有一些情形并不适用，retained fragment 仍是十分有用的（译注：调用 `setRetainInstance(true)` 使 fragment 成为 retained fragment）。举个栗子，如果你从网络服务器或者数据库下载大量数据，retained fragment 在设备旋转时仍持有已下载的数据。

## Additional Code 额外的代码

> When you use fragments to construct your Android application, your project will contain more Java classes and slightly more code than if you did not use fragments at all. Some of this additional code will be used to add a fragment to an activity, which can be tricky if you don’t know how the system works.
> 
> Some Android developers, especially those who have been developing Android applications since before Honeycomb was released, opt to forego using Fragments. These developers will cite the added complexity when using Fragments and the fact that you have to write more code. As I mentioned before, the slight overhead that comes with fragment use is worth the effort.

与不使用 fragments 相比，当你使用 fragments 构建 android app 时，你的项目会包含更多的 Java 类和略多的代码。多出来的部分代码用于把 fragment 添加给 activity，如果你不熟悉其中的机制，会相当棘手。

一些 android 开发者，特别是那些在 Honeycomb 发布前就已经在开发 android app 的，选择放弃使用 fragments。使用 fragments 带来的复杂度和需要写更多的代码是这些开发者的理由。但正如我之前所讲，使用 fragments 需要付出的这点儿努力是值得的。

## Embracing Fragments 涌抱它

> Fragments are a game changer for Android developers. There are a number of significant advantages that come with Fragment-based architectures of Android applications, including optimized experiences based on device size and well-structured, reusable code. Not all developers have embraced fragments, but now is the time. I highly recommend that Android developers make generous use of fragments in their applications. 

对于 Android 开发者而言，fragments 是一个拐点。基于 fragment 构建的应用带来显著的优势，包括基于屏幕尺寸来优化体验、结构良好和可重用的代码。并非所有的开发者涌抱了 fragments，不过现在，时候到溜*（译注：吴吞也是这样说滴）*。我强烈建议 android 开发者大量使用 fragments。

------

译后：安利我自己的几篇关于 Fragment 的总结笔记：

- [如何使用 Android UI Fragment 开发“列表-详情”界面？](http://li2.me/2015/09/how-to-develop-list-detail-ui-with-fragments.html)
- [如何更新及替换 ViewPager 中的 Fragment？](http://li2.me/2015/09/how-to-update-replace-fragment-in-viewpager.html)
- [如何在 Android 设备旋转时暂存数据以保护当前的交互状态？](http://li2.me/2015/11/handling-android-runtime-changes.html)

来来来 让我们一起涌抱Fragment 一起high

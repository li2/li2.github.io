---
layout: post
title: 第一次开发iOS App和Android的对比总结笔记
filename: 2014-12-17-first-iOS-app-summarize.md
category: Android
tags: [android-ios]
---

下述iOS内容，从2014-10-25开始，至2014-12-13，是整个项目中我负责的编码模块，实际的编程时间只有5周，也就是25天的工作日。第一次参与iOS app开发。iOS相关知识储备为0，边写代码边学习，以问题和实现为导向，学的就是快（我觉着快）！

<!-- more -->

1. 实现网格视图，类似于照片预览，iOS可以通过CollectionView实现，android可以通过gird view实现（todo）；

2. 实现多页视图，类似于微信的4个页面，左右滑动和底部的按钮栏都可以切换页面，iOS仍然可以通过CollecView实现，android可以通过View Pager实现；
CollectionView的最小单位是CollectionViewCell，需要向cell填充具体的内容，以完成具体视图的绘制。它的delegate和datasource方法，可以定义cell的个数，可以向当前选中的cell传递数据。
对应到android，
View Pager/Grid View扮演着collection view的功能；
pager adapter扮演着delegate的功能，
fragment扮演着collection view cell的功能；

3. iOS中，把所有涉及UI更新的变量都设置为一个类的property，然后在ViewController和UIView之间传递该类的instance，而且是通过函数参数的方式（目前是这样实现的，应该不是最好的实现方式，todo）；
android中，同样定义了这样一个类，用于存储所有涉及UI更新的变量，在adapter中定义一个public函数，activity调用该函数把数据传递给adapter，然后通过bundle在adaper和fragment之间传递。这又涉及到serialization和parcel两种方法。
（之所以不传递object，有人提到resume后object就消失了，不理解，todo）。

4. 实现列表，类似于聊天app里对话列表，iOS通过picker或table view，android通过listview；
假如想实现点击列表项时，跳转到另一个界面（以展示更详细的内容），
picker有手势gesture（不理解todo），
table view的实现方法，在storyboard中直接建立和destination VC之间的segue指向，然后实现prepare segue method；
android实现方法未知，todo

5. 和列表类似，但不是table似的顺序排列，是多个image或者button在view中分散排布，点击后也需要跳转到另一个界面，
iOS中，绘制这些button的工作，在MVC的架构下，需要在view内完成（不建议在controller内），而当前view又不知道destination controller的内容，所以需要把执行跳转的工作委托给该videw的controller，在source controller中实现delegate方法。source controller通过button的tag属性区分哪一个被按下。
android实现方法未知，todo

6. Third party graph library. iOS中有非常棒的开源的Jawbone chart，android中也有开源的holo graph library。

7. 如何获取view？ iOS可以直接从storyboard拖向code，建立IBOutlet（原理还需要了解 todo）；android通过在layout.xml中为每个元素定义ID，然后通过findviewById获取。

8. 如何添加、布置view？ iOS通过storyboard完成，ios定义了各种constraints（leading、trailing、top、bottom、centerX、centerY），还可以定死高宽（pin heigh、width），通过这些来定义和superview、childview、brother view之间的位置关系。刚开始配置约束条件时，会非常困难，但掌握了技巧，比如等分、比例、把握好参照系，会非常快地绘制界面，极大优势是所见即所得。
android通过layout xml文件，目前掌握的并不好，todo。
对于自定义的view，只需要设置storyboard的属性，在layout中声明该自定义view类的整个package name；

9. 但有些情况下，必须完全用代码programmatically绘制界面，iOS中，比之storyboard，编程方式实现的难点在于定义view自己的位置关系，以谁为参照，因为有些情况下需要remove view。
android实现未知。
两者都可以add veiw to supuer view.

------

**记录以上主题，是为以后整理详细笔记的大纲。**

一份设计，需要在各自在iOS和android平台上实现，**iOS完成了，马上转入android app开发，我只有10天的时间。同样也会遇到很多拦路虎，破之。**
虽然此前有写过一个驱动测试的android app，参与过一个android app的正式开发项目，但我还是没有系统的学习java语言、android平台知识。很匮乏。2014-12-16第一次提交android app代码。

以下是未涉及到的

-----

1. UI架构，我上面实现的模块，只是向这个框架里面添加具体的内容而已。
2. 数据模型建立（这是**最复杂**的部分，需要抽象出涉及到的所有数据为class）。
3. 数据库实现。iOS上通过core manager实现，android上通过sqllite实现。空闲时研究同事实现的代码。
4. 蓝牙连接和数据文件解析（这个**涉及东西就多了**，而且文件解析也特别繁琐，需要把目标格式转成前面定义的data model。。。）
5. 参数设置（也属于界面）。

------

TODO

- iOS公开课
- iOS入门指南（买的那本很厚的很浅的书）、object-c的书
- 设计模式
- java的书（to buy）
- android开发相关书籍（to buy or download）

------

后记

1. 虽然项目时间紧，不仅要实现功能，还得做详尽的测试，我都戴着测试设备睡了好几天；
2. 功能细节上，我擅自实现了目标达成时数字zoom out的动画效果，挺好看的；
3. 我还擅自实现了点击数据图表时，会秀出被点击的bar的具体数值；
4. 我重写了4遍代码，12月份的8个工作日，几乎重写了11月份的所有代码，部分代码重写了4遍，
**重写很大程度上是因为使用了错误的编程方法**，比如使用c风格的pointer to pointer，这就是基础不牢，到处重写。
5. **App开发比驱动维护累**，请注意，我说的是维护驱动，而不是开发驱动。但是挺开心的，说不清楚加班，在某种程度上，
是期望得到大牛同事的肯定，从而获得“这小伙不错，多教他我也乐意”的效果；
还是期望得到boss的首肯以提高加薪的可能性；
还是对知识的渴望、对自己的负责。
我自己都不确定各自比例占多少。但惟一肯定的是，**我很认真！**

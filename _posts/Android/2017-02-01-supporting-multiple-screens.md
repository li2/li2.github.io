---
layout: post
title: Supporting Multiple Screens
description: 
category: Android
tags: [android-view]
---


## 术语和概念


- **Screen size 屏幕尺寸**

    - 又称「屏幕大小」，是屏幕对角线的物理尺寸。

    - 单位英寸 inch，比如 Samsung Note4 是 5.7 英寸。


- **Resolution 屏幕分辨率**


    - 屏幕纵横方向上物理像素的总数，比如 Samsung Note4 是 2560x1440，表示纵向有 2560 个像素，横向有 1440 个像素。

    - pixel 简写 px。


- **Screen density 屏幕密度**


    - 屏幕物理区域中的像素量（quantity of pixels）。

    - 单位 dpi（dots per inch）。 

    - 计算公式：对角线上的像素个数 / 屏幕尺寸，(2560^2 + 1440^2)^(1/2) / 5.7 = 515.3。

    - 六种通用密度：low, medium, high, extra-high, extra-extra-high, and extra-extra-extra-high.

    - **160 dpi 是基线密度**（baseline density、mdpi）。


- **Orientation 方向**


    - 从用户视角看屏幕的方向，即横屏还是竖屏（landscape or portrait），分别表示屏幕的纵横比（screen's aspect ratio）是宽还是高。



- **Density-independent pixel (dp) 密度无关像素**

    - 1dp = 1 px on a 160 dpi screen。

    - dp 单位转换为 px： px = dp * (dpi / 160)，最后一节“不要使用硬编码的像素值”通过代码给出更详细的说明。

    - 应用的 UI 时应始终使用 dp 单位。



## 如何支持多种屏幕


屏幕尺寸和分辨率是用户关心的参数，尺寸越大用户看到的越多、分辨率越高显示越细腻。

屏幕密度是一个物理概念，由屏幕尺寸和分辨率决定。

而 **密度独立性 Density independence**(dp) 是 Android 为了解决屏幕碎片化而抽象出的一个概念，以 **在各种密度的屏幕上保持 UI 元素的物理尺寸**（从用户的视角）。


而对于开发者，应只关注屏幕尺寸和密度：


### 为不同的屏幕尺寸提供替代 layouts


- 特别是横屏或者平板应用，需要调整 UI 元素的位置和尺寸，以利用屏幕空间（比如，竖屏时置于底部的 UI 在横屏时应位于屏幕右侧）；

- ~~*系统提供了 4 种屏幕尺寸限定符：small, normal,large, xlarge，Androd 3.2+ 后被弃用。*~~

- 新**尺寸限定符（size qualifiers）**：smallestWidth `sw<N>dp`, Available screen width `w<N>dp`, Available screen height `h<N>dp`.

- `w<N>dp` 主要用于横屏、多窗格（Multi-pane）；而 `h<N>dp` 很少被用到，因为 UI 垂直滚动，高度更具弹性；`sw<N>dp` 不考虑屏幕方向，只考虑一个最小尺寸。

- 新技术基于布局需要的空间量（the amount of space your layout needs，例如 600dp 宽），而不是尝试让您的布局容纳通用化的尺寸组 （例如大或超大）。在设计 UI 时， 主要关注的可能是 App 在 handset-style UI 与 tablet-style UI that uses multiple panes 之间切换时的实际尺寸。

- **方向限定符（Orientation qualifiers）**：`land` 用于横屏 landscape，`port` 用于竖屏 portrait（默认）。



### 为不同的屏幕密度提供替代 bitmap images


- 系统通过**密度限定符（density qualifiers）**查找匹配的资源目录，包括 ldpi、mdpi、hdpi、xhdpi、xxhdpi 和 xxxhdpi。

- 如果设备屏幕密度是 xxhdpi，那么包含 xxhdpi 限定符（例如 drawable-xxhdpi/）的**密度特定目录（density-specific directory）**可能是最佳匹配项。

- 如果密度特定目录中没有匹配资源，系统不一定使用默认资源（drawable/），而是使用其它密度特定目录进行缩放，但这可能导致模糊、变形。具体是怎样查找的，更多阅读 [《Android 如何查找最佳匹配资源 How Android Finds the Best-matching Resource.》](https://developer.android.com/guide/topics/resources/providing-resources.html#BestMatch)。

- Nine-Patch bitmap file（九宫格位图文件）只拉伸指定的区域。

- 应遵循六种通用密度之间的 **0.75 : 1 : 1.5 : 2 : 3 : 4** 缩放比率，更多阅读 [《Icon Design Guidelines》](https://developer.android.com/guide/practices/ui_guidelines/icon_design.html).

- 这篇文档中有句话，不要感到困惑，因为已经过时了：

    > ~~对启动程序图标以外的 UI 元素不应使用 xxxhdpi 限定符。You should not use the xxxhdpiqualifier for UI elements other than the launcher icon. 引用自《Supporting Multiple Screens》~~
    > ;
    > Choosing to add xxxhdpi versions for the rest of your assets will provide a sharper visual experience（更清晰的视觉体验） on the Nexus 6, but does increase apk size, so you should make an appropriate decision for your app. 引用自 [《Getting Your Apps Ready for Nexus 6 and Nexus 9》2014/10/23](https://android-developers.googleblog.com/2014/10/getting-your-apps-ready-for-nexus-6-and.html)。


### 以适当的 `dp` 值、`wrap_content`、`match_parent` 指定所有布局尺寸值，字体使用 `sp`(scale-independent pixel)



- 上述 size, density qualifiers 也可以用于 `values-<qualifier>`.



### 不要使用硬编码的像素值（hard-coded pixel values）


- Android 系统使用像素作为表示尺寸或坐标值的标准单位，例如 `View.getWidth()` 返回的是 px。

- 将 dp 单位转换为像素单位：

        /**
         * Convert the dps to pixels, based on density scale
         * @param dp value expressed in dps
         * @return value expressed in pixels
         */
        private int dpToPixel(int dp) {
           // Get the screen's density scaling factor
           float scale = getResources().getDisplayMetrics().density;
           // Add 0.5f to round the figure up to the nearest whole number
           return (int) (dp * scale + 0.5f);
        }

    
- 获取 Screen Density (dpi)，例如 Samsung Note4 screen density is 640，这个数字被称为 **quantized density**, 然而屏幕密度的物理值是 515（计算公式见“术语和概念”一节），这个数字被称为 **physical density**。

        getResources().getDisplayMetrics().densityDpi;
    
        weiyi$ adb shell cat /system/build.prop | grep density
    
        weiyi$ adb shell getprop ro.sf.lcd_density
        ro.sf.lcd_density=640


- physical density 就是由物理参数决定的，而 quantized density 是有厂商决定的，这个值一般是屏幕密度分组的「上限值」：120，160，240，320，480，640. quantized density 决定了 **图片缩放系数 Image Scaling Factor (ISF)**：

        
        DisplayMetrics().density
        ISF = ro.sf.lcd_density / 160
        



## 资源



- [《提供资源 Providing Resources》](https://developer.android.com/guide/topics/resources/providing-resources.html)

- [《访问资源 Accessing Resources 》](https://developer.android.com/guide/topics/resources/accessing-resources.html)

- [《Icon Design Guidelines》](https://developer.android.com/guide/practices/ui_guidelines/icon_design.html)

- [《Getting Your Apps Ready for Nexus 6 and Nexus 9》2014/10/23](https://android-developers.googleblog.com/2014/10/getting-your-apps-ready-for-nexus-6-and.html)。



- [How does quantized density affect image resource selection and scaling?](http://stackoverflow.com/questions/30041594/how-does-quantized-density-affect-image-resource-selection-and-scaling)

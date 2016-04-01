---
layout: post
title: 使用 SpannableString 格式化字符串，实现前景色、下划线、超链接、图文混排等
description: 使用 SpannableString 格式化字符串，添加前景色、背景色，改变文字的相对大小，设置粗体、斜体，添加下划线、删除线，设置超链接，添加上标、下标，添加图片到字符串实现图文混排，等等。
category: Android
tags: [android-view]
---

## 格式化字符串的工具类 SpannableStringBuilder

为了格式化字符串，可以使用 [SpannableStringBuilder](http://developer.android.com/reference/android/text/SpannableStringBuilder.html)。
SpannableStringBuilder 有一个方法 `setSpan (Object what, int start, int end, int flags)`，可以把由 start 和 end 指定的部分字符串替换成给定的对象，给定的对象可以是：

- `ForegroundColorSpan` 添加前景色；
- `BackgroundColorSpan` 添加背景色；
- `RelativeSizeSpan` 改变文字的相对大小；
- `StyleSpan` 粗体、斜体等样式；
- `UnderlineSpan` 添加下划线；
- `StrikethroughSpan` 添加删除线；
- `SuperscriptSpan` 上标；
- `SubscriptSpan` 下标；
- `ClickableSpan` 添加 URL 超链接样式；
- `ImageSpan` 添加图片到字符串，实现图文混排（How to add image to text in TextView）；
- 这些 span 继承（或间接继承)自 [CharacterStyle](http://developer.android.com/reference/android/text/style/CharacterStyle.html)。

效果如下图：

![demo](/assets/img/android/android-spannable-string-api.png)


## ImageSpan: 调整图片大小，使图片和 TextView 高度一致

添加 drawable 时，必须动态的获取 TextView 高度，然后调用 `drawable.set(left, top, right, bottom)` 设置 drawable 大小，否则 drawable 和 string 高度不一致，会很难看。
而一般情况下，TextView 的高度被设置为 `wrap_content`，需要在监听器 `ViewTreeObserver.OnGlobalLayoutListener()` 中获取 View 的高度。下面这个函数组合了图片和文本：


```java
private SpannableStringBuilder addImageToText(Context context, int drawableId, String text, int height) {
    Drawable drawable = ContextCompat.getDrawable(context, drawableId);
    int width = height * drawable.getIntrinsicWidth() / drawable.getIntrinsicHeight();
    drawable.setBounds(0, 0, width, height);

    SpannableStringBuilder ssb = new SpannableStringBuilder(" " + text);
    ssb.setSpan(new ImageSpan(drawable), 0, 1, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

    return ssb;
}
```

返回的参数可以直接喂给 `TextView.setText(CharSequence text)`。[完整代码参阅 gist](https://gist.github.com/li2/14ed151eb20c39e4c4b7)。


## ClickableSpan：点击链接调出浏览器

需要两步：

1. 覆写 `ClickableSpan.onClick` 方法；
2. 为包含 ClickableSpan 的 TextView 设置 `MovementMethod`;

```java
    private void setUrlSpanText(TextView textView, final String url) {
        SpannableStringBuilder ssb = new SpannableStringBuilder(CONTENT);
        ssb.setSpan(new ClickableSpan() {
            @Override
            public void onClick(View widget) {
                Toast.makeText(SpannableStringApiActivity.this, url, Toast.LENGTH_LONG).show();
                Intent i = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
                startActivity(i);
            }
        }, START, END, 0);
        textView.setText(ssb);
	
        // setting the MovementMethod on the TextView that contains the span,
        // otherwise onClick will not be called.
        textView.setMovementMethod(LinkMovementMethod.getInstance());
    }
```

完整代码参阅 [Github/SpannableTextActivity.java](https://github.com/TakeoffAndroid/SpannableTextview/blob/master/app/src/main/java/com/takeoffapp/spannabletextview/SpannableTextActivity.java)

## 关于博客
`li2.me` 是我依靠 Jekyll 在 Github 上搭起来的博客。
博客主要内容是技术总结，用来梳理过去学过的东西。以我现在的技术能力还写不出原创技术文章。但从笔记整理、知识梳理的角度讲，我写的文章都是原创的，我对写作的要求是正确、条理、简洁，不做资料的搬运和堆砌。
所以，这是一件挺耗费时间和精力的事情。若无必要，会尽量少写没有含量的二手总结博文，避免误人误己。


## 如何使用Jekyll和Github搭建博客

[这是我第一次搭建时整理的笔记](http://li2.me/2014/05/first-github-jekyll-blog.html)


## 使用的Jekyll主题

目前我使用的主题 fork 自[林安亚的项目](https://github.com/lay1010/lay1010.github.io)，这个模板是我从知乎的一个问答里 [有哪些简洁明快的 Jekyll 模板？](https://www.zhihu.com/question/20223939/answer/29742210) 看到的。它有这些特点：

- 三栏布局：第1栏是分类，第2栏是文章列表或标签列表；第3栏是首页、文章、归档，等其它页面。我经常使用的为知笔记桌面客户端、GitHub桌面客户端也是三栏式布局；
- 颜色搭配：看着非常舒服，我之前也有看到过其它三栏布局，但不喜欢它们的颜色搭配、文章段落、汉字显示的效果；
- 右上角的两个按钮：可以隐藏左边两栏而全屏显示文章；可以显示/隐藏文章目录；

特别喜欢这些，就立刻换掉了我用了很久的主题，并在其基础上做了如下修改：

1. 首页 `index.html` 显示某个分类的文章（而不是标签）；
2. 修改第一栏（分类栏）的 padding 和 fontSize（否则三个汉字就要挤到第二行了）；
3. 在 `_config` 文件中增加全局变量，比如 url_about，其它布局文件通过{{site.url_about}} 方式引用，方便继承这个项目的人修改；
[commit e3d3aed](https://github.com/li2/li2.github.io/commit/e3d3aed75ba3a4c1a91105ea56f2e3e76b457515)
4. 修改第二栏（文章列表栏）的滚动条样式;
[commit 926a1a4](https://github.com/li2/li2.github.io/commit/926a1a4939360417f1bd61be25095f3b8c00ee81)


更详细的内容可以看我这篇文章 [使用三栏式Jekyll主题](http://li2.me/2016/01/%E4%BD%BF%E7%94%A8%E6%96%B0%E4%B8%89%E6%A0%8F%E5%BC%8FJekyll%E4%B8%BB%E9%A2%98.html)

现在我的博客是这个样子：

主页：
![home](/assets/img/util/Jekyll3Theme-Home.png)

标签和归档：
![tag & archives](/assets/img/util/Jekyll3Theme-TagsAndArchives.png)

文章（第一栏和第二栏未隐藏）：
![post with sidebar](/assets/img/util/Jekyll3Theme-PosWithSidebar.png)

文章（第一栏和第二栏已隐藏）：
![post only](/assets/img/util/Jekyll3Theme-PostOnly.png)


------

[之前使用的主题继承自 JekyllPure](https://github.com/li2/JekyllPure)

------

weiyi.li 2016-01-25


## Update 2016-01-28
特别强调： _config.yml 中必须要修改且特别易忽视的配置：

- Disqus shortname 设置
  你必须注册 disqus 账号，申请一个自己的 disqus 子域名（shortname.disqus.com），然后修改 _config.yml 中的 disqus:shortname 配置。这样，你才可以通过 disqus 账号管理自己网站的评论。

- Google analyzing your site's traffic
  你必须注册 google analytics 获取 Tracking ID，然后修改 _config.yml 中的 ga:id 和 ga:url.
  > This property works using Universal Analytics. Click Get Tracking ID and implement the Universal Analytics tracking code snippet to complete your set up.


## Update 2016-02-05
限制 markdown 文本中图片文件的大小：

```css
/* To limit image width, usage: ![book-cover](url) */
img[alt="book-cover"] {
  max-width: 256px;
  display: block;
  margin-left: auto;
  margin-right:auto;
}
```

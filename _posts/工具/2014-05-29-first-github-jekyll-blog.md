---
layout: post
title: 我的第一篇依靠Jekyll搭建在Github上的博客
category: 工具
tags: [github, jekyll]
---

## GitHub使用入门指南

------
**直接在官网学习github命令**
 
- [GitHub Help: Set Up Git](https://help.github.com/articles/set-up-git)
- [GitHub Help: Create A Repo](https://help.github.com/articles/create-a-repo)
- [助你入门 git 的简明指南](http://rogerdudler.github.io/git-guide/index.zh.html)


## GitHub Pages

------
[GitHub Help: 如何创建一个GitHub Pages](https://pages.github.com/)
在GitHub创建一个以`username.github.io`命名的repository；clone到本地；在repository的根目录下新建index.html；push所有的改变。
以上步骤建立了一个静态页面，通过`http://username.github.io`可以看到index.html的内容。

问题是如何更灵活地使用GitHub Pages，像发布博客文章一样？
通过Jekyll。

### Jekyll是什么？

<!-- more -->

以下内容引用自[《像黑客一样写博客——Jekyll入门》by Mort](http://www.soimort.org/posts/101/)

> Jekyll 是一个简洁的、特别针对博客平台的静态网站生成器。它使用一个模板目录作为网站布局的基础框架，并在其上运行 Textile 、 Markdown 或 Liquid 标记语言的转换器，最终生成一个完整的静态Web站点，可以被放置在Apache或者你喜欢的其他任何Web服务器上。它同时也是 GitHub Pages 、一个由 GitHub 提供的用于托管项目主页或博客的服务，在后台所运行的引擎。
> 本中文入门教程由 Mort 基于 Jekyll的官方Wiki 等网页内容翻译整理并维护。

一手资料请参考[Jekyll官网](http://jekyllrb.com/)


### 快速搭建Jekyll

如果你想先跳过Jekyll的细节，可以fork别人的配置，然后commit到自己的GitHub repository`username.github.io`，有了直观的使用体验后再深入细节的学习中，参考：

- [《3分钟建立一个Jekyll Blog》 by Tao Zhang](http://ztpala.com/2012/01/12/zero-to-hosted-jekyll-blog-in-3-minutes/)
- [《使用Github Pages建独立博客》 by BeiYuu](http://beiyuu.com/github-pages/)
- [BeiYuu的Jekyll项目](https://github.com/beiyuu/beiyuu.github.com.git)
- [Ztpala的Jekyll项目](https://github.com/pala/pala.github.com)

[使用find/grep/sed等脚本程序批量重命名Jekyll配置文件中的域名、博客名称](http://stackoverflow.com/questions/6178498/using-grep-and-sed-to-find-and-replace-a-string)
`find /path -type f -exec sed -i 's/oldstr/newstr/g' {} \;`

[通过jekyll创建静态页面](http://jekyllrb.com/docs/pages/)
我使用`/:categories/:title.html`作为permalink，可以为每个分类创建单独的页面。


### 为什么使用GitHub+Jekyll

选择这种方式写博客的原由，你可以听他们讲：

- [《理想的写作环境：Git+Github+Markdown+Jekyll》 by 阳志平](http://www.yangzhiping.com/tech/writing-space.html)
- [《用 Git 维护博客？酷！》 by World Hello](http://www.worldhello.net/2011/11/29/jekyll-based-blog-setup.html)
- 其它搭建博客的方法，你还可以参考[《hexo你的博客》 by 沙烈宝](http://ibruce.info/2013/11/22/hexo-your-blog/)， 通过Hexo+GitHub搭建博客。



## 如果你想使用自己的域名？

------
可以用自己的域名，代替`username.github.io`.
如何修改GitHub配置，具体参考：

- [GitHub Help: Setting up a custom domain with GitHub Pages](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages)
- [GitHub Help: My custom domain isn't working](https://help.github.com/articles/my-custom-domain-isn-t-working)

如何设置域名的DNS解析，具体参考：

- [Godaddy注册商域名修改DNS地址](https://support.dnspod.cn/Kb/showarticle/tsid/42/)
- [Adding or Editing A Records](http://support.godaddy.com/help/article/680/managing-dns-for-your-domain-names)

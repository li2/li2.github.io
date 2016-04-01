---
layout: post
title: GitHub Pages 仅支持 kramdown 后遇到的 markdown 文本渲染问题
description: Github 从 2016.5.1 开始仅支持 `kramdown` 和代码高亮 `rouge`，所以需要修改 `_config.yml` 的 markdown 和 highlighter 配置。替换 markdown 解析器之后导致了一些渲染错误。比如，所有使用 **\`\`\`** 标记的代码块没有被正确渲染；段落内的回车换行符没有被识别。这篇文章罗列了我遇到的这些问题及解决办法。
category: 工具
tags: []
---

Github 从 2016.5.1 开始仅支持 `kramdown`，如果使用了其它 markdown 解析器的话，每天都会收到 github 官方邮件，提醒更改。

> Note: Starting May 1st, 2016, GitHub Pages will only support kramdown. [来源](https://help.github.com/articles/updating-your-markdown-processor-to-kramdown/)

代码高亮也出现了警告：

> To fix page build warnings, you must change your highlighter value to `rouge` in your _config.yml file.[来源](https://help.github.com/articles/page-build-failed-config-file-error/#fixing-highlighting-errors)

既然如此，就按照 Github 要求修改 `_config.yml`：

```xml
markdown: kramdown
highlighter: rouge
```

## 但导致了一个问题：所有使用 **\`\`\`** 标记的代码块没有被正确渲染

解决办法：

> Use the GitHub-Flavored Markdown (GFM) parser / mode. Change your _config.yml settings to:
>    
>        markdown: kramdown
>        kramdown:
>            input: GFM
>            hard_wrap: false
>
> [Refer to: How can I get backtick fenced code blocks (e.g. ```) working (with kramdown)?](https://github.com/planetjekyll/quickrefs/blob/master/FAQ.md#q-how-can-i-get-backtick-fenced-code-blocks-eg--working-with-kramdown)


## 但又导致了另一个问题：段落内的回车换行符没有被识别

也就是说在编辑器内输入的文本：

>    第1行¶
>    第2行¶
>    第3行¶

被渲染成了：

> 第1行 第2行 第3行

这对于你国一个喜欢把文章分成行假装是诗的程序猿来讲，就很尴尬了。

Markdown 需要敲两次回车（有至少一个空白行）才能产生一个新的段落。仅仅一个回车不会被解释为换行。如果想实现段落内换行，需要在行尾**「敲两个空格然后回车」**，或者在行尾加标签 `<br>`，但这两种方式非常不友好，违背了 markdown 引以为傲的简洁特性。

这个问题对此有讨论，支持和反对各有观点：[Should the markdown renderer treat a single line break as \<br\>?](http://meta.stackexchange.com/questions/26011/should-the-markdown-renderer-treat-a-single-line-break-as-br)

怎么解决这个问题呢？把 hard_wrap 设置为 true。其实这个属性默认值就是 true，只是在解决上一个问题时，Github 官网给出的答案设置为了 false，我没研究其含义，就直接复制粘贴了。

> Interprets line breaks literally
> Insert HTML `<br/>` tags inside paragraphs where the original Markdown document had newlines (by default, Markdown ignores these newlines).
> Used by: GFM parser [来源](http://kramdown.gettalong.org/rdoc/Kramdown/Options.html)


## 其它一些问题

Markdown 有很多渲染器，有些扩展了 markdown 的核心语法，所以一份 .md 文档由不同的渲染器可能就会得到不一样的效果。那么，替换渲染器后，这些问题就凸显出来了：

- 表示标题的 `#` 必须与文字内容有 1 个空格隔开：`# Heading`能被正确渲染，而 `#Heading` 可能不会被渲染；
- 区块引用（Blockquotes） `>` 上面要有 1 个空行；
- 列表内的区块引用除了上面的 1 个空行外，还要缩进 4 个空格；
- 代码块每行都需要缩进 4 个空格，当然可以通过一种简洁的方式： \`\`\`，但列表或者区块引用内的代码块，必须要缩进 8 个空格；


我把 Android 分类的文章挨个检查修改了，怎么讲，一个对整洁排版有追求的猿 li21。
还有，不能被一些网站的 markdown 编辑器给惯出坏毛病了。

参考：[Markdown 语法说明 (简体中文版)](http://wowubuntu.com/markdown/index.html)

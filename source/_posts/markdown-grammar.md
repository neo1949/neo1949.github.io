---
title: Markdown 常用语法
date: 2018-05-25 14:21:23
tags:
    - Markdown
categories:
---

本文用于记录 Markdown 的语法。

<!-- more -->
## TODO

## 添加页内跳转（锚点链接）
如果一篇文章很长，没有目录会导致阅读起来很不方便，添加页内跳转（也叫锚点链接）可以解决这个问题。

### 创建锚点
有几种创建锚点的方式，使用哪种方式都可以，只是在显示可能有些差别：
```
[create an anchor](#anchors-in-markdown)
<a id="anchors-in-markdown">create an anchor</a>
<div id="anchors-in-markdown">create an anchor</div>
<span id="anchors-in-markdown">create an anchor</span>
```

这个 <code>anchors-in-markdown</code> 通俗的理解就是一个 <code>key</code>，跳转锚点时就是根据这个 <code>key</code> 来找到对应锚点的，因此这个值必须是唯一的。这个值可以是中文，但必须保证唯一性。通常我们一般都是在章节标题这些地方使用锚点，目的是方便用户快速查看某个章节的内容。


比如：我想给上面的 "添加页内跳转（锚点链接）" 这个章节创建一个锚点，那么可以这样写：
```
[添加页内跳转（锚点链接）](#add_anchor_in_page)
<a id="add_anchor_in_page">添加页内跳转（锚点链接）</a>
<div id="add_anchor_in_page">添加页内跳转（锚点链接）</div>
<span id="add_anchor_in_page">添加页内跳转（锚点链接）</span>
```

### 访问锚点
访问锚点的方式是一样的：
```
[visit an anchor](#anchors-in-markdown)
```

比如我想访问 “添加页内跳转（锚点链接）” 这个锚点，那么就需要这么写：
```
[跳转到 “添加页内跳转（锚点链接）”](#add_anchor_in_page)
```

<code>[]</code> 中的描述内容可以随意些，只要在 <code>#</code> 后面添加想要访问的锚点名称就行了。

###  演示锚点

首先创建四个标题：

#### [标题1](#anchor_titie1)
#### <a id="anchor_titie2">标题2</a>
#### <div id="anchor_titie3">标题3</div>
#### <span id="anchor_titie4">标题4</span>

点击文字跳转到锚点：
点击 [我是标题1的锚点](#anchor_titie1) 跳转到“标题1”小章节。
点击 [我是标题2的锚点](#anchor_titie2) 跳转到“标题2”小章节。
点击 [我是标题3的锚点](#anchor_titie3) 跳转到“标题3”小章节。
点击 [我是标题4的锚点](#anchor_titie4) 跳转到“标题4”小章节。

因为只是演示，所以不会看到明显的界面跳转。在实际使用时，如果要访问的锚点和访问点已经无法在电脑屏幕上同时显示，那么点击访问点可以看到界面会自动跳转到那些已经定义好的锚点。


### 差别
从此页面 <code>hexo</code> 支持的博客渲染效果来看，<code>\[create an anchor\](#anchors-in-markdown)</code> 和 使用 <code>a</code> 标签的创建的锚点，会在锚点的文字描述底部显示一条线 <code>---</code>，表示这是个链接。而使用 <code>div</code> 和 <code>span</code> 标签创建的锚点不会显示链接的标志。因此，为了美观我选择使用 <code>div</code> 和 <code>span</code> 标签创建锚点，具体选择根据个人喜好决定。

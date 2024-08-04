---
title: Wordpress主题开发
date: 2024-07-30 15:26:13
tags: wordpress
categories: 开发经验
---

# wordpress主题开发

## 文件

每个主题至少要有这两个文件 - **style.css** 和 **index.php**。index.php 告诉主题中所有的元素如何布局，style.css 则告诉主题中所有的元素该如何展示，以及它们的样式。下面是一个完整的主题含有的文件列表style.css

- index.php
- home.php
- single.php
- page.php
- archive.php
- category.php
- search.php
- 404.php
- comments.php
- comments-popup.php
- author.php
- date.php、

## WordPress 专业术语：

- **Template（模板**） — 其实就是一个代码集，主题中很多地方会利用到这个代码集，所以把它们整合成一个模板，这样就就不必一遍遍输入这些重复代码。
- **Template file（模板文件**） — 一个包含一个或者多个代码集（模板）文件。每个主题是由多个模板文件组成的，比如：index.php，style.css，sidebar.php 等等。
- **Theme（主题**）或者 **WordPress theme（WordPress 主题）** — 所有你正在使用的文件：文本，图像，代码等等。注意： WordPress theme（主题）和 WordPress template(s)（模板）是两个不同的东西，尽管有些人认为他们一样。
- **Post（日志或者文章）** — 现在你读的就是一篇日志。此外，它是你 blog 的一个简单的条目，如：一个页面或者一篇日记。
- **Page（静态页面）** — 一种特殊的 post，它不是以分类组织的。它有别于你其他的日志。注意：在 WordPress，page（页面）和 Page（静态页面）是两种不同的东西。

WordPress 博客的**每个页面**是**由多个模板文件组成的**，我们可以看出主题的 `index.php` 是由 4 个模板文件组成： `header.php`，`index.php`，`sidebar.php` 和 `footer.php`。

## Header 模板文件:

通常在这个文件中包含博客的**标题**（title）和**描述**（description）。而且它们通常在整个博客中都是一样的。

## Index 模板文件：

这个模板文件包含你的**日志的标题**，**日志的内容**（就是每篇日志的文本和图片）和**日志**的**元数据** （元数据是每篇日志的额外信息，如作者是谁，日志发布的时间，在哪个分类下，有多少留言等等）。



## Sidebar 模板文件

这个模板文件主要用于控制博客的**页面列表**，**类别列表**，**存档列表**，**友情链接列表**和**其他一些列表**。

`footer.php` 通常不会因为页面的改变而改变，你可以在这里放置任何东西，但是通常是**版权信息**。

现在让我解释为什么把上面图片中的 `index.php` 所在的区域标为红色的。引文这块区域是会根据不同类型的页面而发生变化。

如果你在单一日志页面，这时候页面将会包含这四个模板文件：`header.php`，**`single.php`**，`sidebar.php` 和 footer。
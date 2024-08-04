---
title: wordpress优化
date: 2024-07-30 15:29:37
tags: wordpress
categories: 开发经验
---
# wordpress主题

## 安装主题文件大小受限制

- php.ini

  把 upload_max_filesize 和 post_max_size 修改为20M

- /etc/nginx/nginx.conf

  client_max_body_size 20m;

上传主题压缩包安装或者选择网上主题进行安装

## http自动跳转https

```
server {
        listen       80;
        listen       [::]:80;
        server_name  star.amshadow.cn;
		
        #把http的域名请求转成https
		return 301 https://$host$request_uri;
		#强制将http的URL重写成https
        #rewrite ^(.*) https://$server_name$1 permanent;
    }
```

## wordpress插件

### AntiVirus

manual Scan 

### Wordfence安全

扫描

google登录验证

### WPS隐藏登录

### WPS Cleaner

### WPS Bidouille

**加速插件**

WP Fastest Cache


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



# 优化wordpress

先通过top看下信息(shift + m 会让进程按照memory排序)
发现里面有大量php-fpm进程，经过查找资料发现，php-fpm 的 FastCGI 进程一旦加载就不会释放 。因此有必要先针对这个做一个优化。

## php-fpm优化

修改php-fpm文件，一般的地址是/etc/php-fpm.d/www.conf

vi /etc/php-fpm.d/www.conf

修改下面这几个参数，找一下就行(参考文末链接1的内容，结果参数含义如下)

- pm：表示使用那种方式，有两个值可以选择，就是static（静态）或者dynamic（动态），默认为dynamic。
- pm.max_children：静态方式下开启的php-fpm进程数量。
- pm.start_servers：动态方式下的起始php-fpm进程数量。
- pm.min_spare_servers：动态方式下的最小php-fpm进程数量。
- pm.max_spare_servers：动态方式下的最大php-fpm进程数量。

```
pm = dynamic
pm.max_children = 20
pm.start_servers = 5
pm.min_spare_servers = 2
pm.max_spare_servers = 10
pm.max_requests = 300
```

保存，之后，重启php-fpm服务



systemctl restart php-fpm



## 删除没用的主题

删除的方法是就是进入到wordpress的themes目录
比如我的是/usr/share/nginx/html/wordpress/wp-content/themes

cd /usr/share/nginx/html/wordpress/wp-content/themes

然后把theme的文件夹删除就好了， 通过rm

## mysql优化

超乎寻常的有!!! 但是降低一点打开的速度，一般可以降低50%的内存消耗

编辑 my.cnf文件
vi  my.cnf

在文末添加下面这一段 (如果你这个文件中已经有了[mysqld]

[mysqld]
performance_schema = 0

重启mysqld服务
systemctl restart mysqld
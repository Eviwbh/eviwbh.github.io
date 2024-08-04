---
title: Wordpress主题
date: 2024-07-30 15:26:13
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
        server_name  amshadow.cn www.amshadow.cn;
		
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

---
title: typecho
date: 2024-07-30
tags: 
categories: 开发经验
---

## centos安装typecho

```
https://github.com/typecho/typecho
yum -y install unzip

tar -zxvf typecho-1.2.1.tar.gz 

mv typecho-1.2.1/ /usr/share/nginx

cd /usr/share/nginx

mv typecho-1.2.1/ amshadow/

配置ngxin
vi /etc/nginx/servers/amshadow.conf

--------------------------------------------------------
server {
	listen       443 ssl http2;
    listen       [::]:443 ssl http2;
    server_name  amshadow.com www.amshadow.com; 
	root         /usr/share/nginx/amshadow; 
		
		#证书文件名称
        ssl_certificate "/etc/nginx/cert/amshadow.pem";
        #证书私钥文件名称
        ssl_certificate_key "/etc/nginx/cert/amshadow.key";
        #ssl_session_cache shared:SSL:1m;
        #ssl验证配置
        ssl_session_timeout  10m;
        #配置加密套件/算法加密，写法遵循openssl标准
        ssl_ciphers HIGH:!aNULL:!MD5;
        #使用服务器端的首选算法
        ssl_prefer_server_ciphers on;
        #安全链接可选加密协议
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;

	
	    access_log /var/log/nginx/typecho_access.log main;
        if (!-e $request_filename){
            rewrite ^(.*)$ /index.php$1 last;
        }


		location / {
			index index.php index.html index.htm;

			if (!-e $request_filename){
			    rewrite . /index.php last;
			}
		}

	
		location ~ \.php$ {
			fastcgi_pass 127.0.0.1:9000;
			fastcgi_index index.php;
			fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
			fastcgi_split_path_info ^((?U).+\.php)(/?.+)$;
			fastcgi_param PATH_INFO $fastcgi_path_info;
			fastcgi_param SCRIPT_NAME $fastcgi_script_name;
			fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
			include fastcgi_params;
		}
	}

--------------------------------------------------------

chmod -R 777 /usr/share/nginx/amshadow/

mysql -uroot -p
create database typecho_amshadow character set utf8mb4 collate utf8mb4_unicode_ci;


安装主题
cd /usr/share/nginx/amshadow/usr/themes/


https://github.com/HaoOuBa/Joe

unzip Joe-master.zip
```


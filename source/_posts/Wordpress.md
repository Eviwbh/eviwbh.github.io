---
title: Wordpress-CentOS7
date: 2024-07-30 15:26:13
tags: wordpress
categories: 开发经验
---
# Wordpress-CentOS7

centos7安装wordpress

## 安装mysql5.7

```
mkdir /opt/mysql
cd /opt/mysql

yum -y install wget

wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar

tar -xvf mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar

rpm -qa|grep mari  原本可能有的数据库 提前删干净
rpm -e --nodeps mariadb-libs 删除mari相关的所有
rpm -e --nodeps marisa

rpm -ivh mysql-community-common-5.7.26-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.26-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.26-1.el7.x86_64.rpm

 yum -y install libaio
 yum -y install numactl
 
rpm -ivh mysql-community-server-5.7.26-1.el7.x86_64.rpm --force --nodeps 


systemctl start mysqld.service

如果报错Job for mysqld.service failed because the control process exited with error code. See "systemctl status mysqld.service" and "journalctl -xe" for details.

mkdir -p /var/run/mysqld/
chown mysql.mysql /var/run/mysqld/
//or
mkdir /var/lib/mysql/data
vi /etc/my.cnf
datadir=/var/lib/mysql/data


systemctl status mysqld.service

grep "password" /var/log/mysqld.log

mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '@(PdpyY9%X5S4#';
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '@(PdpyY9%X5S4#';


配置远程连接
USE mysql;
UPDATE mysql.user SET host = '%' WHERE user = 'root';
SELECT user, host, plugin, authentication_string FROM mysql.user;
FLUSH PRIVILEGES;

create database star_amshadow_cn character set utf8mb4 collate utf8mb4_unicode_ci;

use wordpress;

quit

配置开机启动启动
查看MySQL是否自启：systemctl is-enabled mysqld
开启自启 ：systemctl enable mysqld
关闭自启 ：systemctl disable mysqld
systemctl enable mysqld.service # 开机自启动
systemctl disable mysqld.service # 禁用开机自启动

//如果服务器内存小可以改
vi /etc/my.cnf
[mysqld]
performance_schema = 0


systemctl restart mysqld
```



## 安装mysql8

```
wget https://repo.mysql.com//mysql80-community-release-el7-9.noarch.rpm

rpm -qa |grep -i mysql

如果有
yum remove -y mysql*
rpm -e --nodeps mysql80 ...

rpm -qa | grep mariadb

如果有
rpm -e --nodeps mariadb-libs-5.5.60-1.el7_5.x86_64

yum install -y libaio

rpm -ivh mysql80-community-release-el7-9.noarch.rpm 

rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023

yum -y install mysql-community-server

vi /etc/my.cnf

#数据库默认字符集, 主流字符集支持一些特殊表情符号（特殊表情符占用4个字节）
character-set-server = utf8mb4

#数据库字符集对应一些排序等规则，注意要和character-set-server对应
collation-server = utf8mb4_general_ci

#设置client连接mysql时的字符集,防止乱码
init_connect='SET NAMES utf8mb4'

#是否对sql语句大小写敏感，1表示不敏感
lower_case_table_names = 1


systemctl start mysqld
systemctl status mysqld

grep "password" /var/log/mysqld.log

mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'password';


配置远程连接
USE mysql;
UPDATE mysql.user SET host = '%' WHERE user = 'root';
SELECT user, host, plugin, authentication_string FROM mysql.user;
FLUSH PRIVILEGES;

create database wordpress character set utf8mb4 collate utf8mb4_unicode_ci;

use wordpress;
quit

配置开机启动启动
查看MySQL是否自启：systemctl is-enabled mysqld
开启自启 ：systemctl enable mysqld
关闭自启 ：systemctl disable mysqld
systemctl enable mysqld.service # 开机自启动
systemctl disable mysqld.service # 禁用开机自启动

mysql -V
```



## 安装Nginx

```
更新yum库
备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak20240409

wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

yum -y install epel-release

yum -y install nginx       
systemctl start nginx       
systemctl enable nginx.service      
```

设置完成后在浏览器输入：公网ip能看到页面则表示安装成功

修改nginx主配置文件
`vi /etc/nginx/nginx.conf`

```
#server {
#		listen 80;
#		server_name shop.richcai.com;
#		return 301 https://server_name$name$request_uri;
#	}


include servers/*;



mkdir /etc/nginx/servers /usr/share/nginx/amshadow_cn /etc/nginx/cert
传好证书

vi /etc/nginx/servers/star_amshadowcn.conf

server {
	    listen       443 ssl http2;
        listen       [::]:443 ssl http2;
        server_name  star.amshadow.cn; 
		root         /usr/share/nginx/star_amshadow_cn; 
		
		#证书文件名称
        ssl_certificate "/etc/nginx/cert/amshadowcn.pem";
        #证书私钥文件名称
        ssl_certificate_key "/etc/nginx/cert/amshadowcn.key";
        #ssl_session_cache shared:SSL:1m;
        #ssl验证配置
        ssl_session_timeout  10m;
        #配置加密套件/算法加密，写法遵循openssl标准
        ssl_ciphers HIGH:!aNULL:!MD5;
        #使用服务器端的首选算法
        ssl_prefer_server_ciphers on;
        #安全链接可选加密协议
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
		
		# Load configuration files for the default server block.
		include /etc/nginx/default.d/*.conf;
	
		location / {
			index index.php index.html index.htm;
			try_files $uri $uri/ /index.php?$args;
		}
	
		rewrite /wp-admin$ $scheme://$host$uri/ permanent;
	
		location ~* ^.+\.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|rss|atom|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$ {
					access_log off; log_not_found off; expires max;
		}
	
		location ~ \.php$ {
			try_files $uri =404;
			fastcgi_split_path_info ^(.+\.php)(/.+)$;
			fastcgi_pass 127.0.0.1:9000;
			fastcgi_index index.php;
			fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
			include fastcgi_params;
		}
	}

------------------------------------------------------------
server {
		listen       80 default_server;
		listen       [::]:80 default_server;
		server_name  39.106.159.62; # 此处修改为域名或者公网IP
		root         /usr/share/nginx/amshadow_cn; # 站点的目录
	
		# Load configuration files for the default server block.
		include /etc/nginx/default.d/*.conf;
	
		location / {
			index index.php index.html index.htm;
			try_files $uri $uri/ /index.php?$args;
		}
	
		rewrite /wp-admin$ $scheme://$host$uri/ permanent;
	
		location ~* ^.+\.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|rss|atom|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$ {
					access_log off; log_not_found off; expires max;
		}
	
		location ~ \.php$ {
			try_files $uri =404;
			fastcgi_split_path_info ^(.+\.php)(/.+)$;
			fastcgi_pass 127.0.0.1:9000;
			fastcgi_index index.php;
			fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
			include fastcgi_params;
		}
	}
```

输入：
`nginx -t -c /etc/nginx/nginx.conf`
检查配置文件
重载：
`systemctl reload nginx`

## 安装php74

```
yum install epel-release

yum -y install http://rpms.remirepo.net/enterprise/remi-release-7.rpm

yum -y install yum-utils

yum install -y php74-php-fpm php74-php-cli php74-php-bcmath php74-php-gd php74-php-json php74-php-mbstring php74-php-mcrypt php74-php-mysqlnd php74-php-opcache php74-php-pdo php74-php-pecl-crypto php74-php-pecl-mcrypt php74-php-pecl-geoip php74-php-recode php74-php-snmp php74-php-soap php74-php-xml --skip-broken

php74 -v

systemctl start php74-php-fpm

systemctl enable php74-php-fpm

reboot
```





## 安装PHP8

```
yum -y install epel-release

yum -y install http://rpms.remirepo.net/enterprise/remi-release-7.rpm

yum -y install yum-utils

yum-config-manager --enable remi-php80

rpm -qa | grep php
yum remove -y php*
rpm -e --nodeps 

yum -y install php php-cli php-common php-devel php-fpm php-gd php-intl php-mbstring php-mysqlnd php-opcache php-pdo php-pecl-apcu php-pecl-imagick php-pecl-mongodb php-pecl-redis php-pecl-xdebug php-pecl-zip php-soap php-xml

php -v

systemctl start php-fpm 

systemctl enable php-fpm    # 设置开机启动

reboot
```



## 测试php-fpm是否安装成功

输入`vi /usr/share/nginx/amshadow_cn/index.php`，按i进入编辑模式，输入以下内容：

```
<?php
    echo "<title>Test Page</title>";
    echo "Hello World!";
?>
```

在浏览器中输入`http：//公网IP/index.php；`
如果浏览器中出现`Hello World!`则表示配置成功,可继续进行以下步骤，若出现文件下载弹窗，则配置失败，检查以上步骤是否出错。



##  安装配置wordpress

```
 wget https://cn.wordpress.org/latest-zh_CN.tar.gz  
 
 tar zxvf latest-zh_CN.tar.gz  
 
 cd wordpress/  
 
 cp wp-config-sample.php wp-config.php 
 
 vi wp-config.php  

/** The name of the database for WordPress */
define('DB_NAME', 'wordpress'); 

/** MySQL database username */
define('DB_USER', 'root');  # 数据库用户名

/** MySQL database password */
define('DB_PASSWORD', '@(PdpyY9%X5S4#');    # 数据库密码

/** Database hostname */
define( 'DB_HOST', 'localhost/ip' );

//文件最后加，不需要FTP
define('FS_METHOD', "direct");


mv * /usr/share/nginx/star_amshadow_cn/ # 将wordpress文件移动web站点的根目录

chown -Rf apache:root /usr/share/nginx/star_amshadow_cn/
```

完成后，在浏览器中输入
`http://你的主机IP或者域名/wp-admin/install.php`
进入到wordpress的配置页面，输入网站标题，用户名和密码后，就可以进入wordpress后台管理界面，到此便大功告成。




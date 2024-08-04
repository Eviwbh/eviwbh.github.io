---
title: gitea
date: 2024-07-30 11:50:33
tags: 
categories: 开发经验
---

# gitea

## 数据库

```
mysql -uroot -p

CREATE USER 'gitea' IDENTIFIED BY 'password';

GRANT ALL PRIVILEGES ON *.* TO 'gitea'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION;

CREATE DATABASE giteadb CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_unicode_ci';
 
GRANT ALL PRIVILEGES ON giteadb.* TO 'gitea';

FLUSH PRIVILEGES;

exit
```

## 安装Gitea

下载二进制包gitea-1.21.4-linux-amd64

https://dl.gitea.com/gitea/1.21.4/gitea-1.21.4-linux-amd64

```
chmod +x gitea-1.21.4-linux-amd64
```

### 添加用户

```
groupadd --system git

adduser \
   --system \
   --shell /bin/bash \
   --comment 'Git Version Control' \
   --gid git \
   --home-dir /home/git \
   --create-home \
   git
```

### 创建工作路径

```
mkdir -p /var/lib/gitea/{custom,data,log}
chown -R git:git /var/lib/gitea/
chmod -R 750 /var/lib/gitea/
mkdir /etc/gitea
chown root:git /etc/gitea
chmod 770 /etc/gitea
```

### 配置Gitea工作路径

```
export GITEA_WORK_DIR=/var/lib/gitea/
```

### 复制二进制文件到全局位置

```
cp gitea-1.21.4-linux-amd64 /usr/local/bin/gitea
```

### 启动Gitea

gitea不允许使用root用户运行

```
su git

GITEA_WORK_DIR=/var/lib/gitea/ /usr/local/bin/gitea web -c /etc/gitea/app.ini
```



## 配置Gitea 

在浏览器输入IP地址+3000端口，访问Gitea的Web界面

云服务器安全组端口3000要打开，反向代理后可以关闭

## gitea关闭用户注册

找到gitea的配置文件gitea/conf/app.ini，把下面的设置改为true即可:

```
ROOT_URL = https://git.eviwbh.com/

DISABLE_REGISTRATION  = true
```



## nginx反向代理

上传ssl证书

```
server {
        listen       443 ssl http2;
        listen       [::]:443 ssl http2;
    server_name git.eviwbh.com;

    location / {
        client_max_body_size 512M;
        proxy_pass http://localhost:3000;
        proxy_set_header Connection $http_connection;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

        #证书文件名称
        ssl_certificate "/etc/nginx/cert/eviwbh_com.pem";
        #证书私钥文件名称
        ssl_certificate_key "/etc/nginx/cert/eviwbh_com.key";
        #ssl_session_cache shared:SSL:1m;
        #ssl验证配置
        ssl_session_timeout  10m;
        #配置加密套件/算法加密，写法遵循openssl标准
        ssl_ciphers HIGH:!aNULL:!MD5;
        #使用服务器端的首选算法
        ssl_prefer_server_ciphers on;
        #安全链接可选加密协议
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
}
```



## gitea自启动

```
vi /etc/rc.local

#gitea
su - git -c "GITEA_WORK_DIR=/var/lib/gitea/ /usr/local/bin/gitea web -c /etc/gitea/app.ini"

chmod +x /etc/rc.d/rc.local
```


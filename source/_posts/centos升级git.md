---
title: centos升级git
date: 2023-04-16
tags: CentOS
---



## centos升级git

### 卸载原有的git

```
rpm -e --nodeps git
```

yum remove git    采用yum删除git，同时也将删除其依赖包

### 安装相关依赖

```
yum install -y curl-devel expat-devel openssl-devel zlib-devel asciidoc

yum install -y gcc perl-ExtUtils-MakeMaker
```

### 下载解压

```
wget https://github.com/git/git/archive/v2.27.0.tar.gz -O git.tar.gz

tar -C /usr/local/src -zxvf git.tar.gz

cd /usr/local/src/git-2.27.0
```



### 编译并设置安装路径

```
vi Makefile
prefix = /usr/local/bin/git

make all

make install
```

### 添加环境变量

```
echo "export PATH=$PATH:/usr/local/bin/git/bin" >> /etc/profile && source /etc/profile

git --version
```
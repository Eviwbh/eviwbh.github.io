---
    title: Hexo
    date: 2022-08-29 23:02:22 
    categories: 开发经验
---


# Hexo 基本操作

- hexo cl
- hexo new post markdown_name
- hexo g
- hexo d
- hexo s

## 上传日志到云服务器 or GitHub

- hexo clean
- hexo g -d

# 安装配置Hexo

## 环境配置

### nvm安装

1. https://github.com/coreybutler/nvm-windows/releases
2. 下载后安装 安装完毕后，找到安装的路径，一些简单配置，打开setting.txt增加镜像源提高下载速度

```none
node_mirror: https://npm.taobao.org/mirrors/node/
npm_mirror: https://npm.taobao.org/mirrors/npm/
```

1. nvm version 检验是否安装成功

### Node安装

1. nvm install 18
2. nvm list
3. nvm use 18 (进入安装目录，用管理员运行cmd)
4. node -v

## 安装hexo

```
npm install hexo-cli -g
```

## 生成SSH Keys

```
ssh-keygen -t rsa -C "eviwbh@163.com"
```



## 创建Hexo

```
cd 目录

hexo init blog(文件名)

cd blog

hexo s

hexo n "markdown_name" == hexo new post markdown_name

```

# 上线博客

## github

账户里配置好之前生成的id_rsa.pub

```
_config.yml中修改deploy

deploy:
  type: git
  repository: https://github.com/Eviwbh/eviwbh.github.io.git
  branch: main
```

## 云服务器

登录云服务器普通用户账号

```
su [username]

mkdir .ssh
vim .ssh/authorized_keys
粘贴之前生成的id_rsa.pub

exit
mkdir 网站存放路径
chown -R [username]:[username] 网站存放路径

su [username]
git init --bare blog.git
vi 网站存放路径/blog.git/hooks/post-receive

#!/bin/bash
git --work-tree=网站存放路径 --git-dir=网站存放路径/blog.git checkout -f


chmod +x 网站存放路径/blog.git/hooks/post-receive
```



```
deploy:
  type: git
  repository: git@ip:网站存放路径/blog.git
  branch: master
```



## 自动部署工具

```
npm install hexo-deployer-git --save
```



# Hexo维护

## hexo其他电脑上写博客

### 原设备

在github上创建分支并设置为默认分支

把新的分支拉下来，删除除了.git 文件夹之外的所有文件

```
git clone ...
```

把之前源码(除了.deploy_git文件夹)都克隆到这个新生成的文件夹中

```
git add .
git commit –m "新大陆"
git push
```

之后就正常更新文章

### 新设备

生成ssh key添加到GitHub账户上

```
ssh-keygen -t rsa -C "email"
```

把仓库的源码拉下来

```
git clone ...
```

安装好环境

```
npm install hexo-cli -g

npm instal
```

之后正常使用

```
hexo clean 
hexo g
hexo d 
hexo s
```



## 多个服务器部署

```
deploy:
  - type: git
    repository: https://github.com/Eviwbh/eviwbh.github.io.git
    branch: main
  - type: git
    repository: git@ip:网站存放路径/blog.git
    branch: master
```





## Hexo去除代码块行号

在配置文件_config.yml中，找到highlight，并将line_number：true改为false。



# 疑难杂症

## Hexo报错hexo : 无法加载文件hexo.ps1，

原因：因为在此系统上禁止运行脚本

### 解决方法设置->隐私和安全性->开发者选项->**允许本地PowerShell脚本在为签名的情况下运行**





## git-receive-pack: command not found



原因：远程服务器上的git安装路径是/usr/local/bin/git，不是默认路径

### 解决方法建立链接文件：

```
ln -s /usr/local/bin/git/bin/git-receive-pack /usr/bin/git-receive-pack
```





## The authenticity of host 'IP' can't be established:ssh



原因：在于每次远程登录Linux的时候，Linux都要检查一下，当前访问的计算机公钥是不是在~/.ssh/know_hosts中，这个文件时OpenSSH记录的。当下次访问相同计算机时，OpenSSH会核对公钥。如果公钥不同，OpenSSH会发出警告，避免你受到DNS Hijack之类的攻击。SSH对主机的public_key的检查等级是根据StrictHostKeyChecking变量来配置的。默认情况下，StrictHostKeyChecking=ask。简单所下它的三种配置值：
1.StrictHostKeyChecking=no
最不安全的级别，当然也没有那么多烦人的提示了，相对安全的内网测试时建议使用。如果连接server的key在本地不存在，那么就自动添加到文件中（默认是known_hosts），并且给出一个警告。
2.StrictHostKeyChecking=ask #默认的级别，就是出现刚才的提示了。如果连接和key不匹配，给出提示，并拒绝登录。
3.StrictHostKeyChecking=yes #最安全的级别，如果连接与key不匹配，就拒绝连接，不会提示详细信息。

#### 解决方法修改云服务器/etc/ssh/ssh_config

```
StrictHostKeyChecking no
```


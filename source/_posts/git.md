---
title: git
date: 2024-07-30 15:15:25
tags: git
categories: 开发工具
---


# 使用github搜索

awesome xxx //找百科全书

xxx sample //找例子

xxx starter /xxx boilerplate //找空项目架子

xxx tutorial //找教程


# gitignore不生效

```
git rm -r --cached .
git add .
git commit -m "update .gitignore"
```


# git基本操作

```
设置查询配置信息
git config --global user.name "govk"
git config --global user.email "china.lijialin@outlook.com"

克隆代码
git clone

查看状态
git status

告诉新建的文件夹要用git管理
git init 

提交代码
//全部提交到缓冲区 暂存区
git add . 
//单次提交 
git add 2.py

提交代码到仓库
git commit -m "加备注" 

推送代码
git push

拉取代码
git pull

查看git提交日志
git log  

回滚
git reset HEAD //reset定向到
git chackout HEAD main.py //回退到main.py最后一次提交的版本
git chackout main.py //没有提交前 从缓冲区退回到工作区

从当前分支为基础新建分支
//创建分支a b
git checkout -b a 
git checkout -b b

切换回master主分支
git checkout master

合并分支a
git merge a
合并有冲突自己难以决策时 放弃当前合并
git merge --abort
看当前有哪些分支
git branch

完成的分支删除
git branch -D a
```



## 连接github

github新建仓库有提示

### …or create a new repository on the command line



```
echo "# -" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/govk/-.git
git push -u origin main
```

### …or push an existing repository from the command line



```
git remote add origin https://github.com/govk/-.git
git branch -M main
git push -u origin main
```

### …or import code from another repository

You can initialize this repository with code from a Subversion, Mercurial, or TFS project.

# vscode

插件下载 gitlens



github最常用的便是SSH和HTTP(S)协议。git关联远程仓库可以使用http协议或者ssh协议。
HTTPS优缺点

优点1: 相比 SSH 协议，可以使用用户名／密码授权是一个很大的优势，这样用户就不必须在使用 Git 之前先在本地生成 SSH 密钥对再把公钥上传到服务器。 对非资深的使用者，或者系统上缺少 SSH 相关程序的使用者，HTTP 协议的可用性是主要的优势。 与 SSH 协议类似，HTTP 协议也非常快和高效
优点2: 企业防火墙一般会打开 80 和 443 这两个常见的http和https协议的端口，使用http和https的协议在架设了防火墙的企业里面就可以绕过安全限制正常使用git，非常方便
缺点: 使用http/https除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令. 但是现在操作系统或者其他git工具都提供了 keychain 的功能，可以把你的账户密码记录在系统里，例如OSX 的 Keychain 或者 Windows 的凭证管理器。

SSH的优缺点

优点1: 架设 Git 服务器时常用 SSH 协议作为传输协议。 因为大多数环境下已经支持通过 SSH 访问 —— 即时没有也比较很容易架设。 SSH 协议也是一个验证授权的网络协议；并且，因为其普遍性，架设和使用都很容易。



缺点1: SSH服务端一般使用22端口，企业防火墙可能没有打开这个端口。
缺点2: SSH 协议的缺点在于你不能通过他实现匿名访问。 即便只要读取数据，使用者也要有通过 SSH 访问你的主机的权限，这使得 SSH 协议不利于开源的项目。 如果你只在公司网络使用，SSH 协议可能是你唯一要用到的协议。 如果你要同时提供匿名只读访问和 SSH 协议，那么你除了为自己推送架设 SSH 服务以外，还得架设一个可以让其他人访问的服务。

总结
HTTPS利于匿名访问，适合开源项目可以方便被别人克隆和读取(但他没有push权限)；毕竟为了克隆别人一个仓库学习一下你就要生成个ssh-key折腾一番还是比较麻烦，所以github除了支持ssh协议必然提供了https协议的支持。
而SSH协议使用公钥认证比较适合内部项目。 当然了现在的代码管理平台例如github、gitliab，两种协议都是支持的，基本上看自己喜好和需求来选择就可以了。


# git将一个项目同时push到多个仓库

使用以下命令添加远程仓库地址

```
git remote set-url --add origin https://github.com/demo.git
```

查看所有远程仓库信息

```
git remote -v
```

此时同步所有仓库

```
git push
```


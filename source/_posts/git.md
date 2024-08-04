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
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

npm install
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


## 上线博客

```
_config.yml中修改deploy

deploy:
  type: git
  repository: https://github.com/Eviwbh/eviwbh.github.io.git
  branch: main
```

下自动部署工具

```
npm install hexo-deployer-git --save
```

## 解决运行Hexo报错hexo : 无法加载文件hexo.ps1，因为在此系统上禁止运行脚本

设置->隐私和安全性->开发者选项->**允许本地PowerShell脚本在为签名的情况下运行**


## Hexo去除代码块行号
在配置文件-config.yml中，找到highlight，并将line_number：true改为false。
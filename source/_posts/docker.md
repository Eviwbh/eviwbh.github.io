---
title: docker
date: 2024-07-30 15:12:50
tags: docker
categories: 开发工具
---
# Docker

## 安装docker

## 容器化和Dockerfile

Dockerfile

```
FROM node:14-alpine //基础镜像，node，v14，基于alpine操作系统
COPY index.js /     //源路径是相对于Dockerfile文件的路径，目标路径是相对于镜像的路径
CMD node /index.js  //可执行程序的名字，可执行程序接收到的参数
```

```
docker build -t hello-docker . //hello-docker是镜像的名字 .表示Dockerfile所在的目录

docker images //查看所有镜像

docker run hello-docker //运行镜像
```


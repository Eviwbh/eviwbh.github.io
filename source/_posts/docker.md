---
title: docker
date: 2024-07-30 15:12:50
tags: docker
categories: 开发工具
---
# Docker

真心不推荐使用

1. 高度集成，仓库消失等于白费，个人最不喜欢把心放在别人那里，自己掌握技术才是最好的。
2. 对技术提升没帮助，直接跳过环境部署会让你对使用的开发软件永远处于小白阶段，自己都做开发了还怕这点麻烦，其实也不麻烦，所谓难者不会，会者不难。

## 启动 Docker

```
sudo systemctl start docker
sudo systemctl restart docker
```

## 镜像管理

```
查看镜像
docker image ls
docker images

检索镜像
docker search [image]
docker search nginx

拉取镜像
docker pull [image]

上传镜像
docker push [image]
docker push hello-docker

保存镜像
docker save [image] -o FILE /
docker save [image] > FILE
docker save hello-docker > hello-docer.tar

导入镜像
docker load -i FILE
docker load -i hello-docer.tar

查看镜像历史
docer history [image]

删除镜像
docker rmi [image]
docker image rm [image]


```



## 容器管理



## 容器运行



# 安装docker

## 卸载旧版本

```
sudo yum remove docker \
        docker-client \
         docker-client-latest \
         docker-common \
         docker-latest \
         docker-latest-logrotate \
         docker-logrotate \
         docker-engine
```

## **设置仓库**

在新主机上首次安装 Docker Engine-Community 之前，需要设置 Docker 仓库。之后，可以从仓库安装和更新 Docker。

```
sudo yum install -y yum-utils
```

### 官方源地址

```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

### 阿里云

```
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

## 安装 Docker Engine-Community

安装最新版本的 Docker Engine-Community 和 containerd，或者转到下一步安装特定版本：

```
sudo yum install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

启动 Docker

```
sudo systemctl start docker
```

# 卸载 docker

删除安装包：

```
yum remove -y docker \
           docker-client \
           docker-client-latest \
           docker-common \
           docker-latest \
           docker-latest-logrotate \
           docker-logrotate \
           docker-selinux \
           docker-engine-selinux \
           docker-engine
```

删除镜像、容器、配置文件等内容：

```
yum remove -y docker-ce \
           docker-ce-cli \
           containerd

rm -rf /etc/systemd/system/docker.service.d
rm -rf /etc/systemd/system/docker.service
rm -rf /var/lib/docker
rm -rf /var/run/docker
rm -rf /usr/local/docker
rm -rf /etc/docker
rm -rf /usr/bin/docker* /usr/bin/containerd* /usr/bin/runc /usr/bin/ctr
```


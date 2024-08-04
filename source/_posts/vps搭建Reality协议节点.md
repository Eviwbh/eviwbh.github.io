---
title: VPS 搭建 Reality协议节点
date: 2023-07-25
tags: shell
categories: 编程语言
---

# VPS 搭建 Reality协议节点

一、**准备工作**
1、VPS一台，重置好系统（CentOS / Debian / Ubuntu）
**本期教程视频使用的 VPS 官网：[点此打开](https://666clouds.com/aff.php?aff=209)**

XTLS官方**Reality**开源仓库：**[点此打开](https://github.com/XTLS/REALITY)**

新版X-UI面板作者开源仓库：**[点此打开](https://github.com/FranzKafkaYu/x-ui/)**

------

**二、开始搭建 Reality**

**1、安装组件**

CentOS系统分别执行如下命令：

```
yum update -y
yum install curl wget -y
```

2、Debian / Ubuntu系统分别执行如下命令

```
apt update -y
apt install curl wget -y
```

------

**三、安装 X-ui 作者开源仓库地址：[点此打开](https://github.com/FranzKafkaYu/x-ui/)**

```
bash <(curl -Ls https://raw.githubusercontent.com/FranzKafkaYu/x-ui/master/install.sh)
```

------

**四、BBR网络拥堵算法**

1、CentOS 开启 BBR 加速

```
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
```

2、Debian / Ubuntu 开启 BBR2 加速

```
wget --no-check-certificate -q -O bbr2.sh "https://github.com/yeyingorg/bbr2.sh/raw/master/bbr2.sh" && chmod +x bbr2.sh && bash bbr2.sh auto
```

------

PS：如搭建完成，无法打开X-UI面板，请检查你VPS端口是否放行，如**Reality**节点搭建好无法使用，请打开网址：[**https://ping.pe**](https://ping.pe/) 输入VPS公网IP查看IP是否被墙！
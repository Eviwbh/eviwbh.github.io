---
title: shell
date: 2024-07-30 11:50:33
tags: shell
categories: 编程语言
---
# shell编程

## shell脚本

- 开头，以什么shell执行 /bash

  #!/bin/bash

- 执行

  - chmod u+x

    ./xxx.sh

  - sh xxx.sh

## 输出echo

echo xxx

echo "xxx"

echo xxx=$变量名

## 系统变量

- HOME $PWD $SHELL $USER等

- 显示当前shell中所有变量 set

## 自定义变量

- 通常变量名大写

- 定义变量：变量名=值

  值：\`函数\`或$(函数)  都能得到函数返回值

  echo a=$(date)

- 输出变量：echo $变量名

- 撤销变量：unset 变量

- 声明静态变量：readonly变量，注意不能unset

## 设置环境变量

vim /etc/profile

- 把shell变量输出成环境变量/全局变量：export 变量名=变量值

- 让修改的配置信息立即生效：source 配置文件

## 多行注释

:<<！

xxxx

xxxx 

！

---
title: Mac
date: 2024-07-30 15:22:11
tags: MacOS
categories: 操作系统
---

# Mac

## 安装

​	sudo spctl --master-disable





### Mac安装MySQL8

```sql
vi ~/.bash_profile

export PATH=${PATH}:/usr/local/mysql/bin/
```

```sql
source ~/.zshrc   # If you use Oh-My-Zsh
source ~/.bashrc  # If you use Default Bash
```



### Mac10.14.6安装Xcode11.3.1

把系统时间设定为距离今日的前一个月或者更多，然后再去解压，注意在解压成功，并install 这段时间里不要把系统时间调回去，等install完成后，再把系统时间还原，否则install会报错。





### Mac安装brew

https://brew.sh/

```
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"

/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```



### Mac安装dart

```
brew tap dart-lang/dart
brew install dart
brew info dart
```

Idea 安装插件 dart



### Mac安装electron

```
npm install -g cnpm --registry=https://registry.npmmirror.com

cnpm install --save-dev electron
```





### Mac安装Flask,requests

进行节点间的通讯

```
pipenv install flask==0.12.2
```

请求和发送包的库

```
pipenv install requests==2.18.4
```





### Mac使用git

```
brew install git
git
git config --global user.name "AsherJw"
git config --global user.email "china_lijialin@outlook.com"
git config --list
```



### Mac安装使用nvm 

```
brew install nvm

echo "source $(brew --prefix nvm)/nvm.sh" >> .bash_profile

. ~/.bash_profile

常用命令
1. nvm list 是查找本电脑上所有的node版本
    - nvm list 查看已经安装的版本
    - nvm list installed 查看已经安装的版本
    - nvm list available 查看网络可以安装的版本
2. nvm install <version> 安装指定版本node
3. nvm use <version> 切换使用指定的版本node
4. nvm ls 列出所有版本
5. nvm current显示当前版本
6. nvm alias <name> <version> ## 给不同的版本号添加别名
7. nvm unalias <name> ## 删除已定义的别名
8. nvm reinstall-packages <version> ## 在当前版本node环境下，重新全局安装指定版本号的npm包
9. nvm on 打开nodejs控制
10. nvm off 关闭nodejs控制
11. nvm proxy 查看设置与代理
12. nvm node_mirror [url] 设置或者查看setting.txt中的node_mirror，如果不设置的默认是 https://nodejs.org/dist/
　　nvm npm_mirror [url] 设置或者查看setting.txt中的npm_mirror,如果不设置的话默认的是： https://github.com/npm/npm/archive/.
13. nvm uninstall <version> 卸载制定的版本
14. nvm use [version] [arch] 切换制定的node版本和位数
15. nvm root [path] 设置和查看root路径
16. nvm version 查看当前的版本
```

### Mac安装node.js

```
nvm install 14.19.3

nvm alias default v

node -v

npm get registry
npm config set registry http://registry.npm.taobao.org
```





### Mac安装pipenv

pip使用来安装python包的工具

pipenv为每一个项目提供一个单独的python运行环境

```
pip3 install pipenv
使用
pipenv --python=python3
```



### Mac安装配置Java环境

安装jdk后

```
vim .bash_profile

export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_281.jdk/Comtents/Hom
e

java -version
```





## 问题



### Mac关闭chrome更新提示

```
cd ~/Library/Google 
 
sudo chown root:wheel GoogleSoftwareUpdate
```



### Mac解决系统更新提示

终端输入：

```
sudo softwareupdate --ignore "macOS Mojave”

defaults write com.apple.systempreferences AttentionPrefBundleIDs 0

killall Dock
```

#### 强制永久解决方案

如果想永久生效，我们必须把每次开机加载的系统自动更新进程干掉才行。

​			此方案采用禁止权限的思路，在终端输入以下命令，强制吊销系统更新通知权限。但是这样会禁用系统更新的自启权限，阻碍正常的系统更新通知，请慎用。

```
sudo mount -uw /

sudo chmod 600 /System/Library/PrivateFrameworks/SoftwareUpdate.framework/Versions/A/Resources/SoftwareUpdateNotificationManager.app/Contents/MacOS/SoftwareUpdateNotificationManager

defaults write com.apple.systempreferences AttentionPrefBundleIDs 0

killall Dock
```

注意提示:Operation not permitted，解决方式是

1. 重启同时按cmd+r 在恢复模式中打开终端， 执行 csrutil disable 关闭SIP功能
2. 再次正常重新启动，就可以执行「sudo mount -uw /」 和后面的命令了
3. 执行完之后，别忘记再次重启进入恢复模式，执行csrutil enable，把SIP功能打开

#### 系统自动更新

如果你已经禁止了系统自动更新提示进程的权限，又后悔了，想要恢复，只需要输入以下命令即可：

```
sudo mount -uw /

sudo chmod 777 /System/Library/PrivateFrameworks/SoftwareUpdate.framework/Versions/A/Resources/SoftwareUpdateNotificationManager.app/Contents/MacOS/SoftwareUpdateNotificationManager

defaults write com.apple.systempreferences AttentionPrefBundleIDs 0

killall Dock
```





## 知识

### 查看ip

```
ifconfig
```



### Mac的SIP系统完整性

#### 查看是否开启

​		打开终端
​			csrutil status
​		已打开
​			System Integrity Protection status: enabled.
​		已关闭
​			System Integrity Protection status: disabled

#### 开关

1. 重启电脑，开机过程中按住 Command + R 组合键，直到出现苹果Logo
2. 点击「实用工具」，选择「终端」
   	csrutil disable
      			csrutil enable

### 启用root用户

​	用户群租，登陆选项，网络账户服务器，打开实用工具，top栏编辑，启用root用户

### 文件操作

​	文件路径复制
​		option+command+c
​	文件路径搜索
​		shift+command+g
​	文件详情
​		command+i
​	隐藏文件
​		shift+command+.

### 输入法

​	翻页选词：{ }
​	中英文数字混合输入：shift+字母 option+数字
​	拆字输入：输入拆字后 shift+空格
​	标志声调：添加ABC扩展，在这种输入法下 先输入option+AEV～是四个声调 ā
​	Emoji：command+空格+control😍

### 三指拖拽

​	辅助功能，鼠标触控板，触控板选项，启动拖拽，三指拖拽

### 任意来源

​	sudo spctl --master-disable



### 删除

1、使用键盘快捷键**Command+Delete**将直接删除文件而不用确认，使用该快捷键删除的文件可以从废纸篓恢复。

2、使用键盘快捷键**Command+Option+Delete**立即删除文件。使用此快捷方式删除选定文件而不将其发送到废纸篓。该项目将立即被删除，无法撤消。

3、使用键盘快捷键**Command+Shift+Delete清空废纸篓。**使用此快捷方式清空垃圾箱文件夹。

4、**功能 (Fn) + Delete –**此键允许您删除光标后或光标右侧的文本（向前删除）。

5、**Control + D –**如果您的键盘没有Delete键，请使用此组合。这将删除光标后的文本（向前删除）。

6、**功能 (Fn)+Option +Delete-**此组合键删除光标后的整个单词（仅单词）（向前删除）。

### 隐藏文件

```
defaults write com.apple.finder AppleShowAllFiles -boolean false;killall Finder
defaults write com.apple.finder AppleShowAllFiles -boolean true;killall Finder
```

### Mac终端设置



#### zsh引入bash_profile

```
执行vi ~/.zshrc打开.zshrc,将 source ~/.bash_profile 粘贴到最下面，保存即可
```

#### 终端隐藏计算机名称

```
若shell为zsh

sudo vi /etc/zshrc

# PS1="%n@%m %1~ %# "
PS1="%1~ %# "

:wq!

若shell为bash

sudo vim /etc/bashrc

# PS1='\h:\W \u\$ '
PS1='\W \$ '

:wq!
```

#### Shell更改为Bash

```
chsh -s /bin/bash

chsh -s /bin/zsh
```





# Mac tips  

## 签名  

### xcode-select --install  

### sudo codesign --force --deep --sign -  

## 输入法  

### 翻页选词：{ }  

### 中英文数字混合输入：shift+字母  

option+数字  

### 拆字输入：输入拆字后 shift+空格  

### 标志声调：添加ABC扩展，在这种输入法下 先输入option+AEV～是四个声调 ā  

### Emoji：command+空格+control😍  

## 任意来源  

### sudo spctl --master-disable  

## 解决系统更新提示  

### 终端输入：  

#### sudo softwareupdate --ignore "macOS Mojave”  

#### defaults write com.apple.systempreferences AttentionPrefBundleIDs 0  

#### killall Dock  

### 强制永久解决方案  

#### 如果想永久生效，我们必须把每次开机加载的系统自动更新进程干掉才行。  

##### 此方案采用禁止权限的思路，在终端输入以下命令，强制吊销系统更新通知权限。但是这样会禁用系统更新的自启权限，阻碍正常的系统更新通知，请慎用。  

##### sudo mount -uw /  

##### sudo chmod 600 /System/Library/PrivateFrameworks/SoftwareUpdate.framework/Versions/A/Resources/SoftwareUpdateNotificationManager.app/Contents/MacOS/SoftwareUpdateNotificationManager  

##### defaults write com.apple.systempreferences AttentionPrefBundleIDs 0  

##### killall Dock  

#### 注意  

##### 提示:Operation not permitted，解决方式是  

###### 1. 重启同时按cmd+r  在恢复模式中打开终端， 执行 csrutil disable 关闭SIP功能  

###### 2. 再次正常重新启动，就可以执行「sudo mount -uw /」  和后面的命令了  

###### 3. 执行完之后，别忘记再次重启进入恢复模式，执行csrutil enable，把SIP功能打开  

#### 如果你已经禁止了系统自动更新提示进程的权限，又后悔了，想要恢复，只需要输入以下命令即可：  

##### sudo mount -uw /  

##### sudo chmod 777 /System/Library/PrivateFrameworks/SoftwareUpdate.framework/Versions/A/Resources/SoftwareUpdateNotificationManager.app/Contents/MacOS/SoftwareUpdateNotificationManager  

##### defaults write com.apple.systempreferences AttentionPrefBundleIDs 0  

##### killall Dock  

## 文件操作  

### 文件路径复制  

#### option+command+c  

### 文件路径搜索  

#### shift+command+g  

### 文件详情  

#### command+i  

### 隐藏文件  

#### shift+command+.  

## SIP系统完整性  

### 查看是否开启  

#### 打开终端  

##### csrutil status  

#### 已打开  

##### System Integrity Protection status: enabled.  

#### 已关闭  

##### System Integrity Protection status: disabled  

### 开关  

#### 重启电脑，开机过程中按住 Command + R 组合键，直到出现苹果Logo  

#### 点击「实用工具」，选择「终端」  

##### csrutil disable  

##### csrutil enable  

## Java环境配置  

### 第一次配置环境变量  

#### touch .bash_profile  

### open -e .bash_profile  

### JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home  

PATH=$JAVA_HOME/bin:$PATH:.  
CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.  
export JAVA_HOME  
export PATH  
export CLASSPATH  

### source .bash_profile  

### echo $JAVA_HOME  

## 三指拖拽  

### 辅助功能，鼠标触控板，触控板选项，启动拖拽  

## Mac启用root用户  

### 用户群租，登陆选项，网络账户服务器，打开实用工具，top栏编辑，启用root用户  


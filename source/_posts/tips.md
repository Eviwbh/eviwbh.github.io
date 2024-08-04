---
title: 软件使用技巧
date: 2024-07-30
tags: 
categories: 工具
---

# 软件使用技巧



## 批量重命名文件、文件夹、拓展名Everything

Everything是一款好用的**文件搜索定位工具**，能够帮助我们在电脑中快速找到想要的文件，体积小巧，界面简洁易用。

此外，它也可以**批量重命名文件、文件夹、拓展名**。

打开软件后，找到**“搜索——高级搜索”**，找到我们想要重命名的文件所在文件夹。

选择部分文件或全选文件后，按**F2键**，会弹出重命名的框，可以直接在新表达式中输入，也可以选择系统识别到的规则表达式输入，之后点击**“确定”**即可。



## CMD:

### 内网发送信息

```
MSG /server:内网ip * "找你没事情，下午出去玩吧"
```

### 查看当前所有连接过的wifi密码

```
netsh wlan show profiles

netsh wlan show profile name="" key=clear
```

### 查看ip和自己的外网地址

```
ipconfig /all
curl -L ip.tool.lu
```

### 远程控制

```
mstsc
```

### 修改字体颜色

```
color all
color a
永久修改
regedit编辑查找DefaultColor
数据数值改成a
```

### 查看电脑上的用户防入侵

```
net user
net user name /del
```

### 电脑卡检查系统完整性

```
sfc /SCANNOW
```



## 网络ip

```
cip.cc
```



## typora设置图片相对路径

![image-20230408172522710](img/image-20230408172522710.png)



![image-20230408173829879](img/image-20230408173829879.png)





## pr打不开 报错

```
Application Specific Information:
C:\Program Files\Adobe\Adobe Premiere Pro CC 2018\ZXPSignLib-minimal.dll



Thread 0 Crashed:	[18148]	THREAD_PRIORITY_NORMAL
0	ZXPSignLib-minimal  0x00000000056473e3 ? Unknown - (Symbols generated from a DLL export table)	[  Error:487 试图访问无效的地址。 ]
1	 ???? (  Error: 126 找不到指定的模块。 ) 0x00000000527a4d77 ? Unknown - ()	[  Error:126 找不到指定的模块。 ]
```

把pr中的文件复制到安装目录替换
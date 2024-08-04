---
title: Winsodws
date: 2024-07-30 15:24:24
tags: Windows
categories: 操作系统
---
# Windows

## 安装



### C语言图形库Easyx安装

安装easyx

网易云音乐狗都不用，从现在开始退坑了，害我找了半个小时错误

```none
#include <graphics.h>
#include <MMSystem.h> //播放音乐的头文件,不能放在最上面
#include <stdlib.h>

#pragma comment(lib,"winmm.lib")//pragma编程 用来加载winmm.lib库文件

int main(void)
{
	//创建一个图形界面
	initgraph(600, 400);
	//0表示把图片加载到默认窗口，默认输出设备是控制台
	loadimage(0,"bg.jpg");//项目字符集改成使用多字节字符集

	//播放音乐
	//mciSendString("open bg.mp3 alias song", NULL, 0, NULL);
	 mciSendString("play bg1.mp3 repeat",0,0,0);//如返回277可能是MP3的解码器出了问题，下载解码器解码MP3，finalcodecs
        										  //网易云下载的音乐虽然是MP3格式的但是在这个函数是播放不了的，因为下载的音乐其实是flac格式，所以播放不了，要在qq音乐下载才可以播放。
	
	system("pause");//把窗口暂停查看运行
	
	closegraph();//把当前窗口关闭

	return 0;
}
```





### nvm安装 

- 目录不要有空格
- cmd要用管理员启动 
  - nvm list
  - Nvm  install
  - Nvm  use





## 问题

### Office激活错误

#### Error Code: 0x80080005

解决方法
office甚至没有提供0x80080005错误码的含义，最后在Office-Tool的Issues中找到了解决方法：
https://github.com/YerongAI/Office-Tool/issues/216#issue-836891168

删除注册表中的 KEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\SppExtComObj.exe, 重新激活即可


#### 清除office激活秘钥方法

1、CMD管理员身份运行
2、找到OFFICE安装目录C:\Program Files (x86)\Microsoft Office\Office16 回车
3、cmd窗口内运行CD C:\Program Files (x86)\Microsoft Office\Office1 6回车
4、cmd窗口运行cscript ospp.vbs /dstatus 回车，查看激活秘钥秘钥后5位
5、cmd窗口运行cscript ospp.vbs /unpkey:XXXXX 回车（X代表秘钥后5位）
6、cmd窗口运行cscript ospp.vbs /remhst 回车，刷新激活状态
7、cmd窗口运行cscript ospp.vbs /dstatus 回车，重新查看激活状态
若有多个秘钥需要卸载可重复上述步骤



### Hexo报错

解决运行Hexo报错hexo : 无法加载文件hexo.ps1，因为在此系统上禁止运行脚本

设置->隐私和安全性->开发者选项->**允许本地PowerShell脚本在为签名的情况下运行**





## 知识

### 查看ip

```
ipconfig
```

### 查看端口占用

```
查看占用端口
netstat -aon|findstr "port"  

查看占用端口号的程序
tasklist|findstr "PID"   

杀死占用端口程序 
taskkill /F /IM 进程名.exe
taskkill /F /PID PID
```



### 添加启动自动运行应用

1. 选择“**开始**”按钮 ，然后滚动查找你希望在启动时运行的应用。
2. 右键单击该应用，选择**“更多”**，然后选择**“打开文件位置”**。此操作会打开保存应用快捷方式的位置。如果没有“**打开文件位置**”选项，这意味着该应用无法在启动时运行。
3. 文件位置打开后，按 **Windows 徽标键** + **R**，键入“**shell:startup**”，然后选择“**确定**”。这将打开“**启动**”文件夹。
4. 将该应用的快捷方式从文件位置复制并粘贴到**“启动”**文件夹中。



### 光标移动快捷键

1、基本操作快捷键
Ctrl + Z：  撤销
Ctrl + Y：  重做
Ctrl + S：  保存
Ctrl + F：  查找
Ctrl + F3：  查找上一个
F3：  查找下一个
Ctrl + R：  替换

2、光标选中快捷键
Shift + ← / → (方向键)：  光标向左/向右选中一个字符
Ctrl + Shift + ← / → (方向键)：  光标向左/向右选中一个单词
Shift + Home：  光标从当前位置一直选中到行首
Shift + End：  光标从当前位置一直选中到行尾

3、光标跳转快捷键
Home：  光标跳转至行首
End：   光标跳转至行尾
PgUp：  光标跳转至首行
PgDn：  光标跳转至末行
Ctrl + ← / → (方向键)：  光标向左/向右跳转一个单词
Ctrl + Home：  光标跳转至首行首个字符前面(文件开头)
Ctrl + End：  光标跳转至末行最后一个字符后面(文件结尾)



### capsLock键切换中英文

使用[autohotkey](https://www.autohotkey.com/) 做了热键配置

.ahk文件代码如下：

```
SetStoreCapslockMode, off
CapsLock::
If StartTime
    return
StartTime := A_TickCount
return

CapsLock up::
TimeLength := A_TickCount - StartTime
if (TimeLength >= 1 and TimeLength < 200)
{
    Send, ^{Space}
}
else if (TimeLength >= 200)
{
    Send, {CapsLock}
}
StartTime := ""
return
```

上述代码效果是：长按CapsLock键锁定大写,敲击CapsLock进行切换中英文（替代了ctrl+空格的效果）



设置步骤：

1：下载并安装autohotkey；

2：将中英文切换按键设置改为ctrl+space（即ctrl+空格），shift切换勾选去掉；

3：保存上述代码为.ahk文件设置为开机启动（以管理员执行）；

以上。



### VSCode 添加右键菜单命令

#### 第一步：win+R 输入regedit，打开注册表编辑器

#### 第二步：添加右键文件VSCode打开命令

1. 依次找到 HKEY_CLASSES_ROOT > * > shell > VSCode(若无手动新建.项),点击VSCode文件夹，双击右侧默认在弹出对话框的数值数据选项中键入命令名称，如 Open with VSCode
2. 添加命令图标，右键VSCode文件夹，新建字符串值，双击修改新值名称为 Icon(首字母大写i)，数值数据选项粘贴上VSCode文件所在位置路径，如 D:\develop\VSCode-win32-x64-1.46.1\Code.exe
3. 右键VSCode，新建项，名称为 command ,双击默认，在数值数据中键入VSCode文件所在位置+空格+"%1" 如 D:\develop\VSCode-win32-x64-1.46.1\Code.exe "%1"
4. 此时右键文件查看命令是否好用，是否有图标

#### 第三步：添加右键文件夹VSCode打开命令

1. 依次找到 HKEY_CLASSES_ROOT > Directory > shell > VSCode
2. 添加操作同上

#### 第四步：添加右键桌面VSCode打开命令

1. 依次找到 HKEY_CLASSES_ROOT > Directory > Background > shell > VSCode
2. 添加操作同上
3. 注意：command的数值数据中空格后面要用"%V"



### 关闭Win10自动更新

#### 一、禁用Windows Update服务

1、同时按下键盘 Win + R，打开运行对话框，然后输入命令 services.msc ，点击下方的“确定”打开服务。

2、找到 Windows Update 这一项，并双击打开。

![img](https://pics1.baidu.com/feed/18d8bc3eb13533fa65d18d5de591ad1541345b2e.png?token=5679f417ec3f74e955366f07f5b3fa81)

3、双击打开它，点击“停止”，把启动类型选为“禁用”，最后点击应用。

4、接下再切换到“恢复”选项，将默认的“重新启动服务”改为“无操作”，然后点击“应用”“确定”。



#### 二、在组策略里关闭Win10自动更新相关服务

1、同时按下Win + R 组合快捷键打开运行命令操作框，然后输入“gpedit.msc”，点击确定。

2、在组策略编辑器中，依次展开 计算机配置 -> 管理模板 -> Windows组件 -> Windows更新

![img](https://pics1.baidu.com/feed/738b4710b912c8fcd0c0d4dbb341c24fd6882144.jpeg?token=63e910e43f428dcf23760c49324d822f)



3、然后在右侧“配置自动更新”设置中，将其设置为“已禁用”并点击下方的“应用”然后“确定”。

4、之后还需要再找到“删除使用所有Windows更新功能的访问权限”，选择已启用，完成设置后，点击“应用”“确定”。

#### 三、禁用任务计划里边的Win10自动更新

1、同时按下 Win + R 组合快捷键打开““运行”窗口，然后输入“taskschd.msc”，并点击下方的“确定”打开任务计划程序。

![img](https://pics3.baidu.com/feed/63d9f2d3572c11dfeb1fcd582c6532daf603c24f.jpeg?token=4c91e61fce0787f0dad1854886f6597f)



2、在任务计划程序的设置界面，依次展开 任务计划程序库 -> Microsoft -> Windows -> WindowsUpdate，把里面的项目都设置为 [ 禁用 ] 就可以了。(我这里边只有一个任务，你的电脑里可能会有2个或者更多，全部禁用就行了)



#### 四、在注册表中关闭Win10自动更新

1、同时按下 Win + R 组合快捷键，打开运行对话框，然后输入命名 regedit，然后点击下方的「 确定 」打开注册表。

2、在注册表设置中，找到并定位到 [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\UsoSvc]。然后在右侧找到“Start”键。



![img](https://pics0.baidu.com/feed/8b13632762d0f70311a7aca259b801372697c507.jpeg?token=d9592ed1b88fc4a7b831788c77cb43a0)



3、点击修改，把start值改成16进制，值改为“4”，然后点击「 确定 」保存数据

4、继续在右侧找到“FailureActions”键，右键点击修改该键的二进制数据，将“0010”、“0018”行的左起第5个数值由原来的“01”改为“00”，完成后，点击下方的“确定”即可。



![img](https://pics6.baidu.com/feed/4034970a304e251f1f964138eac4991d7e3e535d.jpeg?token=4a212f600037b02b30da036a68d0a65d)



至此彻底关闭win10自动更新的操作步骤就全部完成了





### git上传图片到GitHub 得到cdn连接加速

- git clone ssh链接需要电脑公钥 //关联GitHub库
- 上传图片文件
- git status //查看状态
- git add
- git commit -m ‘提交信息’
- git push
- 在GitHub上发布版本
- https://cdn.jsdelivr.net/gh/user/库名@版本号/文件路径
- 文件不能超过50M
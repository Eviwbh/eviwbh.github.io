---
title: Linux
date: 2024-07-30 15:18:43
tags: Linux
categories: 操作系统
---

# Linux

## 知识

### 开机自启动

```
vi /etc/rc.local

#gitea
su - git -c "GITEA_WORK_DIR=/var/lib/gitea/ /usr/local/bin/gitea web -c /etc/gitea/app.ini"

chmod +x /etc/rc.d/rc.local
```



### 解压缩

#### gzip/gunzip 指令

gzip 用于压缩文件， gunzip 用于解压的

gzip 文件 （功能描述：压缩文件，只能将文件压缩为*.gz 文件）

gunzip 文件.gz （功能描述：解压缩文件命令）

案例 1: gzip 压缩， 将 /home 下的 hello.txt 文件进行压缩

gzip /home/hello.txt

案例 2: gunzip 压缩， 将 /home 下的 hello.txt.gz 文件进行解压缩

gunzip /home/hello.txt.gz

#### zip/unzip 指令

zip 用于压缩文件， unzip 用于解压的，这个在项目打包发布中很有用的

zip [选项] XXX.zip 将要压缩的内容（功能描述：压缩文件和目录的命令）

unzip [选项] XXX.zip （功能描述：解压缩文件）

zip 常用选项

-r：递归压缩，即压缩目录

unzip 的常用选项

-d<目录> ：指定解压后文件的存放目录

应用实例

案例 1: 将 /home 下的 所有文件/文件夹进行压缩成 myhome.zip

zip -r myhome.zip /home/ [将 home 目录及其包含的文件和子文件夹都压缩] 案例 2: 将 myhome.zip 解压到 /opt/tmp 目录下

mkdir /opt/tmp

unzip -d /opt/tmp /home/myhome.zip

#### tar 指令

tar 指令 是打包指令，最后打包后的文件是 .tar.gz 的文件。

tar [选项] XXX.tar.gz 打包的内容 (功能描述：打包目录，压缩后的文件格式.tar.gz)

| 选项 | 功能               |
| :--- | :----------------- |
| -e   | 产生.tar打包文件   |
| -v   | 显示详细信息       |
| -f   | 指定压缩后的文件名 |
| -z   | 打包同时压缩       |
| -x   | 解包.tar文件       |

案例 1: 压缩多个文件，将 /home/pig.txt 和 /home/cat.txt 压缩成 pc.tar.gz tar -zcvf pc.tar.gz /home/pig.txt /home/cat.txt

案例 2: 将/home 的文件夹 压缩成 myhome.tar.gz tar -zcvf myhome.tar.gz /home/

案例 3: 将 pc.tar.gz 解压到当前目录

tar -zxvf pc.tar.gz

案例4: 将myhome.tar.gz 解压到 /opt/tmp2 目录下 (1) mkdir /opt/tmp2 (2) tar -zxvf /home/myhome.tar.gz -C /opt/tmp2



### 静态库和动态库

https://blog.csdn.net/Pxx520Tangtian/article/details/122931769



### 帮助指令

#### man 获得帮助信息

基本语法：man [命令或配置文件]（功能描述：获得帮助信息）

#### help 指令

基本语法：help 命令 （功能描述：获得 shell 内置命令的帮助信息）



### 软链接 ln 指令

软链接也称为符号链接，类似于 windows 里的快捷方式，主要存放了链接其他文件的路径
ln -s [原文件或目录] [软链接名] （功能描述：给原文件创建一个软链接）

案例 1: 在/home 目录下创建一个软连接 myroot，连接到 /root 目录

ln -s /root /home/myroot

案例 2: 删除软连接 myroot

rm /home/myroot

当我们使用 pwd 指令查看目录时，仍然看到的是软链接所在目录。



### 搜索文件和文件中的内容

#### find

find 目录名 -name 文件名 -print

从/tmp目录开始搜索，把全部的*.c文件显示出来。

find /tmp -name *.c -print

| 选项  | 功能                             |
| :---- | :------------------------------- |
| -name | 按照指定的文件名查找模式查找文件 |
| -user | 查找属于指定用户名的所有文件     |
| -size | 按照指定的囚犯们大小查找文件     |

案例 1: 按文件名：根据名称查找/home 目录下的 hello.txt 文件

find /home -name hello.txt

案例 2：按拥有者：查找/opt 目录下，用户名称为 nobody 的文件

find /opt -user nobody

案例 3：查找整个 linux 系统下大于 200M 的文件（+n 大于 -n 小于 n 等于, 单位有 k,M,G）
find / -size +200M

#### locate 

yum -y install mlocate

locate 指令可以快速定位文件路径。locate 指令利用事先建立的系统中所有文件名称及路径的 locate 数据库实现快速定位给定的文件。Locate 指令无需遍历整个文件系统，查询速度较快。为了保证查询结果的准确度，管理员必须定期更新 locate 时刻

updatedb

locate 搜索文件

由于 locate 指令基于数据库进行查询，所以第一次运行前，必须使用 updatedb 指令创建 locate 数据库。

案例 1: 请使用 locate 指令快速定位 hello.txt 文件所在目录

#### which 

可以查看某个指令在哪个目录下，比如 ls 指令在哪个目录

which ls

#### grep 和 管道符号 |

grep 过滤查找 ， 管道符，“|”，表示将前一个命令的处理结果输出传递给后面的命令处理。

grep [选项] 查找内容 源文件

-n 显示匹配行及行号

-i 忽略字母大小写

案例 1: 请在 hello.txt 文件中，查找 “yes” 所在行，并且显示行号

写法 1: cat /home/hello.txt | grep “yes”

写法 2: grep -n “yes” /home/hello.txt



### 显示文本文件的内容

#### cat

cat 文件名

#### more

more 文件名

按空格键显示下一页，按b键显上一页，按q键退出。

#### tail

tail 文件 （功能描述：查看文件尾 10 行内容）

tail -n 5 文件 （功能描述：查看文件尾 5 行内容，5 可以是任意行数）

tail -f 文件 （功能描述：实时追踪该文档的所有更新）

#### head

head 文件 (功能描述：查看文件头 10 行内容)

head -n 5 文件 (功能描述：查看文件头 5 行内容，5 可以是任意行数)

#### less

less 文件名

| 操作        | 功能说明                       |
| :---------- | :----------------------------- |
| 空格        | 向下翻动一页                   |
| pagedown    | 向下翻动一页                   |
| pageup      | 向上翻动一页                   |
| /或者？字符 | 向下搜索字串，n向下找，N向上找 |
| q           | 离开less这个程序               |

#### wc

统计文本文件的行数、单词数和字节数

wc 文件名

文件行数 文件字数 文件大小



### linux修改ssh端口

#### centos修改ssh端口

```
vi /etc/ssh/sshd_config

Port 端口号

systemctl restart sshd
```

#### debian修改ssh端口

```
vi /etc/ssh/sshd_config

Port 端口号

service ssh restart 
```

#### ubuntu修改ssh端口

```
vi /etc/ssh/sshd_config

Port 端口号

service ssh restart 
```





### RPM、YUM、wget

#### rpm 包的管理

rpm 用于互联网下载包的打包及安装工具，它包含在某些 Linux 分发版中。它生成具有.RPM 扩展名的文件。RPM 是 RedHat Package Manager（RedHat 软件包管理工具）的缩写，类似 windows 的 setup.exe，这一文件格式名称虽然打上了 RedHat 的标志，但理念是通用的。

Linux 的分发版本都有采用（suse,redhat, centos 等等），可以算是公认的行业标准了。

##### rpm 包的简单查询指令

查询已安装的 rpm 列表

```
rpm –qa|grep xx
```

举例： 看看当前系统，是否安装了 firefox 指令: rpm -qa | grep firefox

##### rpm 包名基本格式

一个 rpm 包名：firefox-60.2.2-1.el7.centos.x86_64 名称:firefox

版本号：60.2.2-1

适用操作系统: el7.centos.x86_64

表示 centos7.x 的 64 位系统

如果是 i686、i386 表示 32 位系统，noarch 表示通用

##### rpm 包的其它查询指令：

rpm -qa :查询所安装的所有 rpm 软件包

rpm -qa | more

rpm -qa | grep X [rpm -qa | grep firefox ]

rpm -q 软件包名 :查询软件包是否安装案例：rpm -q firefox

rpm -qi 软件包名 ：查询软件包信息案例: rpm -qi firefox

rpm -ql 软件包名 : 查询软件包中的文件比如： rpm -ql firefox

rpm -qf 文件全路径名 查询文件所属的软件包

rpm -qf /etc/passwd rpm -qf /root/install.log

##### 卸载rpm 包：

- 基本语法

rpm -e RPM 包的名称 //erase

- 应用案例

删除 firefox 软件包

rpm -e firefox

- 细节讨论

1. 如果其它软件包依赖于您要卸载的软件包，卸载时则会产生错误信息。如： $ rpm -e foo

removing these packages would break dependencies:foo is needed by bar-1.0-1

1. 如果我们就是要删除 foo 这个 rpm 包，可以增加参数 –nodeps ,就可以强制删除，但是一般不推荐这样做，因为依赖于该软件包的程序可能无法运行。如：$ rpm -e –nodeps foo

##### 安装rpm 包

- 基本语法

rpm -ivh RPM 包全路径名称

- 参数说明

i=install 安装

v=verbose 提示

h=hash 进度条

- 应用实例

演示卸载和安装 firefox 浏览器

rpm -e firefox

rpm -ivh firefox

#### yum

Yum 是一个 Shell 前端软件包管理器。基于 RPM 包管理，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包。

##### yum 的基本指令

查询 yum 服务器是否有需要安装的软件

yum list|grep xx 软件列表

##### 安装指定的yum 包

yum install xxx 下载安装

yum -y install xxx 自动同意询问

##### yum 应用实例：

案例：请使用 yum 的方式来安装 firefox

rpm -e firefox

yum list | grep firefox

yum install firefox

#### wget 

wget 链接 下载文件



### ubuntu系统ssh服务

终端输入ifconfig查看ip

1. 进入root用户模式：
2. 执行命令： apt-get install ssh（安装ssh服务）
3. apt-get install vim（安装vim编辑器）
4. vi /etc/ssh/sshd_config (更改SSH服务的配置文件，允许远程登录root)
5. 将PermitRootLogin 的值设为 yes



### ubuntu终端下MySQL输入中文

1. 编辑mysql.cnf文件
   首先编辑/etc/mysql/conf.d/mysql.cnf文件,应当无内容

2. 编辑mysqld.cnf文件

vi /etc/mysql/mysql.conf.d/mysqld.cnf

在[mysqld]下加入character-set-server=utf8

3. 重启服务器
   sudo /etc/init.d/mysql restart



### ubuntu的默认root密码设置

一、Ubuntu的默认root密码是随机的，即每次开机都有一个新的root密码。可以在终端输入命令 sudo passwd，然后输入当前用户的密码，enter.

二、终端会提示输入新的密码并确认，此时的密码就是root新密码。修改成功后，输入命令 su root，再输入新的密码就ok了。



### 找回Linux root密码

1. 启动系统，进入开机界面，在界面中按“e”进入编辑界面。
2. 进入编辑界面，使用键盘上的上下键把光标往下移动，找到以““Linux16”开头内容所在的行数”，在行的最后面输入：init=/bin/sh。
3. 输入完成后，直接按快捷键：Ctrl+x 进入**单用户模式**。
4. 在光标闪烁的位置中输入：mount -o remount,rw /（注意：各个单词间有空格），完成后按键盘的回车键（Enter）。
5. 在新的一行最后面输入：passwd， 完成后按键盘的回车键（Enter）。输入密码，**然后再次确认密码即**可, 密码修改成功后，会显示passwd…..的样式，说明密码修改成功
6. 在鼠标闪烁的位置中（最后一行中）输入：touch /.autorelabel（注意：touch与 /后面有一个空格），完成后按键盘的回车键（Enter）
7. 继续在光标闪烁的位置中，输入：exec /sbin/init（注意：exec与 /后面有一个空格），完成后按键盘的回车键（Enter）,等待系统自动修改密码，完成后，系统会自动重启, **新的密码生效**了



### Linux运行级别

#### 基本介绍

运行级别说明：

0 ：关机

1 ：单用户【找回丢失密码】

2：多用户状态没有网络服务

3：多用户状态有网络服务

4：系统未使用保留给用户

5：图形界面

6：系统重启

常用运行级别是 3 和 5 ，也可以指定默认运行级别

命令：init [0123456] 通过 init 来切换不同的运行级别

#### centos7 后运行级别说明

在 centos7 以前， /etc/inittab 文件中 .

进行了简化 ，如下:

```none
multi-user.target: analogous to runlevel 3 
graphical.target: analogous to runlevel 5 
# To view current default target, run: 
systemctl get-default
# To set a default target, run:
systemctl set-default TARGET.target
```



### Linux目录结构

在 Linux 世界里，一切皆文件

- /bin [常用] (/usr/bin 、 /usr/local/bin) 是 Binary 的缩写, 这个目录存放着最经常使用的命令

- /sbin (/usr/sbin 、 /usr/local/sbin) s 就是 Super User 的意思，这里存放的是系统管理员使用的系统管理程序。

- /home [常用] 存放普通用户的主目录，在 Linux 中每个用户都有一个自己的目录，一般该目录名是以用户的账号命名

- /root [常用] 该目录为系统管理员，也称作超级权限者的用户主目录

- /lib 系统开机所需要最基本的动态连接共享库，其作用类似于 Windows 里的 DLL 文件。几乎所有的应用程序都需要用到这些共享库

- /lost+found 这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件

- /etc [常用] 所有的系统管理所需要的配置文件和子目录, 比如安装 mysql 数据库 my.conf

- /usr [常用] 这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似与 windows 下的 program files 目录。

- /boot [常用] 存放的是启动 Linux 时使用的一些核心文件，包括一些连接文件以及镜像文件
- /proc [不能动] 这个目录是一个虚拟的目录，它是系统内存的映射，访问这个目录来获取系统信息
- /srv [不能动] service 缩写，该目录存放一些服务启动之后需要提取的数据
- /sys [不能动]这是 linux2.6 内核的一个很大的变化。该目录下安装了 2.6 内核中新出现的一个文件系统 sysfs =》【别动】
- /tmp 这个目录是用来存放一些临时文件的

- /dev 类似于 windows 的设备管理器，把所有的硬件用文件的形式存储

- /media [常用] linux 系统会自动识别一些设备，例如 U 盘、光驱等等，当识别后，linux 会把识别的设备挂载到这个目录下

- /mnt [常用] 系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将外部的存储挂载在/mnt/上，然后进入该目录就可以查看里的内容了。 d:/myshare

- /opt 这是给主机额外安装软件所存放的目录。如安装 ORACLE 数据库就可放到该目录下。默认为空
- /usr/local [常用]这是另一个给主机额外安装软件所安装的目录。一般是通过编译源码方式安装的程序

- /var [常用]这个目录中存放着在不断扩充着的东西，习惯将经常被修改的目录放在这个目录下。包括各种日志文件

- /selinux [security-enhanced linux]SELinux 是一种安全子系统,它能控制程序只能访问特定文件, 有三种工作模式，可以自行设置.



## 安装



### centos安装使用Ftp

#### 安装ftp

在CentOS7中，采用yum来安装ftp软件包，包括ftp服务器和ftp客户端。如果已经安装，再次执行yum就会把软件包升级到最新版本。

##### 1、安装ftp服务器

yum -y install vsftpd

##### 2、安装ftp客户端

yum -y install ftp

#### 配置ftp服务器

ftp的传输模式有被动模式和主动式两种，缺省是被动模式，主动模式的应用场景极少，为了方便表达，在接下来的内容中只介绍被动模式，主动模式在本文中也有介绍。

##### 1、关闭SELINUX

```
vi /etc/selinux/config

SELINUX =disabled

setenforce 0
```

##### 2、配置ftp数据端口参数

ftp的数据端口也称为高端口，在/etc/vsftpd/vsftpd.conf文件中配置，由pasv_min_port和pasv_max_port两个参数指定，如果文件中没有这两个参数，手工的加进去。

```
vi /etc/vsftpd/vsftpd.conf

listen=YES 

listen_ipv6=NO

pasv_address=<EIP>
pasv_min_port=5000  # 高端口范围的最小值。
pasv_max_port=5500  # 高端口范围的最大值。
```

##### 3、开通防火墙

开通防火墙的方法有两种：

1）开通ftp服务。

firewall-cmd --zone=public --add-service=ftp --permanent

2）开通ftp服务需要的端口，21是控制端口，5000-5500是数据端口范围，也就是上一节中在/etc/vsftpd/vsftpd.conf文件中配置的pasv_min_port和pasv_max_port参数。

firewall-cmd --zone=public --add-port=21/tcp --permanent

firewall-cmd --zone=public --add-port=5000-5500/tcp --permanent

重启防火墙：

systemctl restart firewalld.service

##### 4、启动vsftpd服务

ftp服务器的服务名是vsftpd，相关的操作如下：

```
systemctl start  vsftpd  # 启动服务。

systemctl stop  vsftpd  # 停止服务。

systemctl restart vsftpd  # 重启服务。

systemctl status vsftpd  # 查看服务状态。

systemctl enable vsftpd  # 启用开机自动动vsftpd服务。

systemctl disable vsftpd  # 禁用开机自动动vsftpd服务。
```

##### 5、云平台访问策略配置

如果您购买的是云服务器上，需要登录云服务器提供商的管理平台开通访问策略（或安全组），开通21和高端口的访问策略。

不同云服务器提供商的管理平台操作方法不同，具体操作方法阅读操作手册、或者百度，或者咨询云服务器提供商的客服。

如果云服务器的ftp服务不能建立数据会话，在百度中输入**“被动模式下FTP不能建立数据会话问题“**可以找到解决问题的方法，目前的阿里云服务器就存在这个问题。

#### ftp开发

示例：

```
vi ftpclient.cpp


#include  "_ftp.h"

Cftp ftp;

int main()
{
  if (ftp.login("127.0.0.1:21","lijialin","mima")==false)
  {
    printf("ftp.login(127.0.0.1:21) failed.\n"); return -1;
  }

  printf("ftp.login(127.0.0.1:21) success.\n");

  if (ftp.mtime("/home/lijialin/text.cpp")==false)
  {
    printf("ftp.mtime(/home/lijialin/text.cpp) failed.\n"); return -1;
  }
  printf("ftp.mtime(/home/lijialin/text.cpp) success,mtime=%s.\n",ftp.m_mtime);

  if (ftp.size("/home/lijialin/text.cpp")==false)
  {
    printf("ftp.size(/home/lijialin/text.cpp) failed.\n"); return -1;
  }
  printf("ftp.size(/home/lijialin/text.cpp) success,size=%d.\n",ftp.m_size);

  if (ftp.nlist("/home/lijialin","/tmp/lijialin.lst")==false)
  {
    printf("ftp.nlist(/home/lijialin) failed.\n"); return -1;
  }
  printf("ftp.nlist(/home/lijialin) success.\n");

  if (ftp.get("/home/lijialin/text.cpp","/tmp/text.cpp.bak",true)==false)
  {
    printf("ftp.get() failed.\n"); return -1;
  }
  printf("ftp.get() success.\n");

/*
  if (ftp.put("/root/ftpclient.cpp","/home/zhangkun/bbb/ftpclient.cpp",true)===
false)
  {
    printf("ftp.put() failed.\n"); return -1;
  }
  printf("ftp.put() success.\n");
*/

  ftp.logout();

  return 0;
}
```



```
g++ -g -o ftpclient ftpclient.cpp /project/public/_ftp.cpp /project/public/_public.cpp -I/project/public -L/project/public -lftp -lm -lc
```

#### 动态链接库问题

```
./ftpclient: error while loading shared libraries: libftp.so: cannot open shared object file: No such file or directory

export LD_LIBRARY_PATH=/project/public:.
```



```
ftp.nlist(/home/lijialin) failed. 


```





### centos安装使用nginx

```
sudo yum -y install nginx 

nginx  
ps -ef|grep nginx

yum install lsof
lsof -i:80

nginx -s signal // quit/stop/reload/reopen
```

#### 配置文件

```
nginx -V  
//--conf-path=/etc/nginx/nginx.conf 配置文件位置
//--prefix=/usr/share/nginx   nginx根目录

nginx -t //检查配置文件内容是否正确
nginx -s reload 

vi /etc/nginx/nginx.conf

//全局块 配置worker进程的数量、指定运行服务的用户等
worker_processes auto; //配置worker进程的数量


//events块 配置服务器和客户端网络连接的配置
events {
    worker_connections 1024; //每个worker进程可以接收网络连接的数量
}


//http块     配置虚拟主机（server块）、 反向代理、负载均衡等等  
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types; 	//包含其他配置文件
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80; //监听端口
        listen       [::]:80;
        server_name  _;	//服务名
        root         /usr/share/nginx/html; //配置根目录文件夹

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
    
    include servers/* //把servers目录下的所有配置文件都包含进来，每个虚拟主机（server块）的配置放在一个单独的文件里 
} 
```

#### 网站部署

例如：Hexo，基于Node.js的博客框架

```
npm install hexo-cli -g

mkdir ~/nginx
cd ~/nginx
hexo init blog

cd blog/public
cp -rf * /usr/share/nginx/html //把public下的文件都复制到nginx配置的server的根目录中

hexo使用：
npm install hexo-cli -g
hexo init blog
hexo s //运行一下
hexo d //自动部署 需要配置config文件

```

例如：Vue 

可以修改配置文件server块实现一个服务器代理多个域名访问

```
npm create vite
cd vite-project
npm install
//打包一下
npm run build

把打包后的dist文件放在一个有权限访问的位置 在servers文件夹中添加配置文件 name.conf 写上server块

server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/dist;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
    
nginx -s reload

vite使用：
npm create vite
npm install
npm run build
npm run dev //运行一下

```



#### 反向代理、负载均衡

```
vi /etc/nginx/nginx.conf

在http块中添加反向代理配置
upstream name357{
	ip_hash;	//这个负载均衡策略会对客户端的ip进行哈希，同一个客户端的请会分配到同一个服务器上可以解决一些session的问题
	//另一种负载均衡策略weight
	server 127.0.0.1（ip）:8000 weight=5 //这个服务器性能好，就配置权重（weight）大点，能分配到的请求次数越多
	server 127.0.0.1（ip）:8001
	server 127.0.0.1（ip）:8002
}

访问localhost/app 就相当于 轮流（默认）或负载均衡（weight分配的权重） 的访问对应ip服务器的对应8000/8001/8002端口得到的内容
server{
	location /app{//所有以app开头的请求都被代理到upstream中访问8000，8001......
		proxy_pass http://name357  //地址要和upstream中的名字保持一致
	}
}
```

#### HTTPS

http+ssl=https

ssl证书可以通过云平台获取

在nginx中配置

```
vi /etc/nginx/nginx.conf

在server中 
server {
        listen       443 ssl http2;
        listen       [::]:443 ssl http2;
        server_name  url; //网站域名
        root         /usr/share/nginx/html;

		#证书文件名称
        ssl_certificate "/root/nginx/cacert.pem";
        #证书私钥文件名称
        ssl_certificate_key "/root/nginx/private.key";
        #ssl_session_cache shared:SSL:1m;
        #ssl验证配置
        ssl_session_timeout  10m;
        #配置加密套件/算法加密，写法遵循openssl标准
        ssl_ciphers HIGH:!aNULL:!MD5;
        #使用服务器端的首选算法
        ssl_prefer_server_ciphers on;
        #安全链接可选加密协议
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```





### centos安装git

```
yum -y install git

git --version
```

### centos安装使用nvm

```
git clone git://github.com/creationix/nvm.git ~/nvm
or
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash

command -v nvm

echo "source ~/nvm/nvm.sh" >> ~/.bashrc
source ~/.bashrc


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

### centos安装node.js

```
nvm install 16

node -v

```





### centos7升级GCC

https://blog.csdn.net/b_ingram/article/details/121569398

```
yum install -y gcc gcc-c++
gcc --version
g++ --version
切换用户：
su - root
安装centos-release-scl：
sudo yum install -y http://mirror.centos.org/centos/7/extras/x86_64/Packages/centos-release-scl-rh-2-3.el7.centos.noarch.rpm

sudo yum install -y http://mirror.centos.org/centos/7/extras/x86_64/Packages/centos-release-scl-2-3.el7.centos.noarch.rpm

安装devtoolset：
sudo yum install -y devtoolset-9-gcc-c++

激活对应的devtoolset:
scl enable devtoolset-9 bash
source /opt/rh/devtoolset-9/enable
gcc --version
1.直接替换旧版本的gcc:
mv /usr/bin/gcc /usr/bin/gcc-4.8.5
ln -s /opt/rh/devtoolset-9/root/bin/gcc /usr/bin/gcc
2.启动gcc（永久）
echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
```



### centos安装g++

```
yum install gcc-c++ libstdc++-devel 
```



### centos安装tomcat

```
mkdir /opt/tomcat

apache-tomcat-8.5.59.tar.gz上传到/opt/tomcat

cd /opt/tomcat
tar -zxvf apache-tomcat-8.5.59.tar.gz
cd apache-tomcat-8.5.59/bin/
./startup.sh
```

开放端口（防火墙或安全组） 8080

```
firewall -cmd --permanent add-port=8080/tcp
firewall -cmd --reload
firewall -cmd --query-port=8080/tcp
```



### ubuntu安装mysql

1、#查看有没有安装MySQL：

dpkg -l | grep mysql

2、 安装MySQL：

sudo apt install mysql-server

3、检查是否安装成功：

netstat -tap | grep mysql

### centos安装mysql5.7

```none
mkdir /opt/mysql
cd /opt/mysql

wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar

tar -xvf mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar

rpm -qa|grep mari  原本可能有的数据库 提前删干净
rpm -e --nodeps mariadb-libs 删除mari相关的所有
rpm -e --nodeps marisa

rpm -ivh mysql-community-common-5.7.26-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.26-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.26-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.26-1.el7.x86_64.rpm --force --nodeps 
 如果失败 可能缺少依赖numactl 
 yum -y install libaio
 yum -y install numactl

systemctl start mysqld.service

如果报错Job for mysqld.service failed because the control process exited with error code. See "systemctl status mysqld.service" and "journalctl -xe" for details.

mkdir -p /var/run/mysqld/
chown mysql.mysql /var/run/mysqld/
//or
mkdir /var/lib/mysql/data
vi /etc/my.cnf
datadir=/var/lib/mysql/data


systemctl status mysqld.service

grep "password" /var/log/mysqld.log

mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'password';


配置远程连接
USE mysql;
UPDATE mysql.user SET host = '%' WHERE user = 'root';
SELECT user, host, plugin, authentication_string FROM mysql.user;
FLUSH PRIVILEGES;

quit

配置开机启动启动
查看MySQL是否自启：systemctl is-enabled mysqld
开启自启 ：systemctl enable mysqld
关闭自启 ：systemctl disable mysqld
systemctl enable mysqld.service # 开机自启动
systemctl disable mysqld.service # 禁用开机自启动


```

#### mysql找回root密码

```none
1. vi /etc/my.cnf
2. skip-grant-tables
3. service mysqld restart
4. mysql -u root -p
5. show databases;
6. use mysql;
7. show tables;
8. desc user;
9. update user set authentication_string=password("lijialin")  where user='root'; 
 或者改成空密码''登录以后再修改
10. flush privileges;
11. exit
12. vi /etc/my.cnf
13. #skip-grant-tables
14. service mysqld restart
15. mysql -u root -p
```

#### 连接mysql

```
mysql -u root -p
use mysql;
update user set host = '%' where user = 'root';
FLUSH PRIVILEGES;
```

```none
1.ping IP 查看是否开放公网 
2.telnet IP port 查看端口是否开放
上面不通畅登录进入服务器登录mysql root
如果是docker安装
docker exec -it mysql /bin/bash  执行docker容器的mysql，这里容器名是mysql
mysql -u root -p
3.use mysql;
4. select host,user from user;
5. grant all privileges on *.* to 'root'@'%' identified by 'lijialin'; //5.7
 GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION; //8.0
6. flush privileges;

```

#### 一、查看3306端口是否开放 

```perl
netstat -an|grep 3306
```

如果看到下图这样的，说明端口并未打开：

 

![img](https://img-blog.csdn.net/20170811100345823?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZnJlZXppbmd4dQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


#### 二、修改访问权限

```undefined
cd /etc/mysql/mysql.conf.d/
sudo vim mysqld.cnf
# 
```

```r
# bind-address = 127.0.0.1
```

#### CentOS ‘mysql/mysql.h‘: No such file or directory

yum install mysql-devel

然后

mysql_config

查看-lmysqlclient这个库 在哪个目录

然后只需要把之前的编译命令- lmysqlclient替换成 -L/usr/lib64/mysql -lmysqlclient（目录）这个就好了，就能编译过了。

意思是 不从默认的文件夹/usr/lib里边取动态库-lmysqlclient从绝对目录里边取。

所以也可以分析出来出现这个 找不到这个库的原因不是 因为linux机器上没有这个库。

而是 找错了 位置，如果嫌加绝对目录太麻烦，直接复制一份到/usr/lib一份就行了



### centos安装 JDK

```
mkdir /opt/jdk

jdk-8u261-linux-x64.tar.gz上传到 /opt/jdk 下

cd /opt/jdk
tar -zxvf jdk-8u261-linux-x64.tar.gz
mkdir /usr/local/java
mv /opt/jdk/jdk1.8.0_261 /usr/local/java
vim /etc/profile

export JAVA_HOME=/usr/local/java/jdk1.8.0_261
export PATH=$JAVA_HOME/bin:$PATH

source /etc/profile
java -version
```



### centos安装Hexo

```
cd nginx

npm install hexo-cli -g

hexo init blog

cd blog;npm install

hexo server / hexo s
```



### ubuntu安装redis

```
apt update
```

```
apt install redis-server
```

### centos安装redis

```
gcc -v
yum install -y gcc

cd /opt
# 下载，我是在root下执行的下载，所以我的下载目录为：/root/redis-6.2.6，这里按照自己的实际情况调整
wget https://download.redis.io/releases/redis-6.2.6.tar.gz
# 解压
tar -zxvf redis-6.2.6.tar.gz
# 进入解压目录
cd redis-6.2.6
# 编译
make

yum install tcl tcl-devel -y
make test

make install 

通过守护进程方式启动
vi redis.conf
# 修改内容如下：
daemonize 的值从 no 修改成 yes

redis-server redis.conf

ps -ef |grep redis
redis-cli

设置开机自动启动
cd /lib/systemd/system/
vi redis.service


redis.service
--------------------------------------------------------
[Unit]
Description=redis-server
After=network.target


[Service]

Type=forking

# ExecStart需要按照实际情况修改成自己的地址

ExecStart=/usr/local/bin/redis-server /opt/redis-6.2.6/redis.conf

PrivateTmp=true


[Install]

WantedBy=multi-user.target

--------------------------------------------------------

# 开机自动启动
systemctl enable redis.service
# 启动redis服务
systemctl start redis.service
# 查看服务状态
systemctl status redis.service
# 停止服务
systemctl stop redis.service
# 取消开机自动启动(卸载服务)
systemctl disabled redis.service
```



### ubuntu安装webbench

```
sudo apt-get install exuberant-cTags
wget  http://home.tiscali.cz/cz210552/distfiles/webbench-1.5.tar.gz
tar -zvxf webbench-1.5.tar.gz
cd webbench-1.5
make && sudo make install
webbench --version
```

使用

用法：webbench -c 100 -t 10 http://www.iteye.com/ 

其中：
-c表示并发数，
-t表示时间(秒)

 注意url结尾一定要加上/



### ubuntu16.04 安装boost1.65

参考：https://blog.csdn.net/jiandanjinxin/article/details/65632653

#### 1. 下载源码

https://www.boost.org/users/history/version_1_65_1.html

```
tar -zxvf boost_1_65_1.tar.gz
cd boost_1_65_1
./bootstrap.sh --with-python=PYTHON
./bootstrap.sh –with-libraries=all
会生成 b2和bjam文件。可查看使用方法
./bootstrap.sh -help
```

#### 2. 编译 boost

```
sudo ./b2 install 
./b2 cxxflags=-fPIC cflags=-fPIC --c++11
```





## 问题



### ubuntu系统“软件中心”闪退或者打不开解决方法

```
sudo apt-get update
sudo apt-get dist-upgrade
sudo apt-get install --reinstall software-center
```



### killall: command not found

#### 解决：

centos 执行如下安装命令：

```
yum install psmisc
```



### CentOS ‘mysql/mysql.h‘: No such file or directory

#### 解决：

yum install mysql-devel

然后

mysql_config

查看-lmysqlclient这个库 在哪个目录

然后只需要把之前的编译命令- lmysqlclient替换成 -L/usr/lib64/mysql -lmysqlclient（目录）这个就好了，就能编译过了。

意思是 不从默认的文件夹/usr/lib里边取动态库-lmysqlclient从绝对目录里边取。

所以也可以分析出来出现这个 找不到这个库的原因不是 因为linux机器上没有这个库。

而是 找错了 位置，如果嫌加绝对目录太麻烦，直接复制一份到/usr/lib一份就行了





## 命令



### linux共享内存

#### 查看当前共享内存

ipcs -m

#### 删除指定共享内存

ipcrm -m 32575(shmid)



### 进程操作

#### 查看进程

```
ps -ef |grep procctl(程序名)
```

#### 开机启动进程

```
用root登录 看看有没有这个文件
ls -l /etc/rc.d/rc.local

vi /etc/rc.d/rc.local

需要开机启动什么直接在后面加，例如
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.

touch /var/lock/subsys/local

#启动守护进程
/project/tools1/bin/procctl 30 /project/tools1/bin/checkproc

#启动后台服务程序
# su -lijialin -c "/bin/sh /project/idc1/c/start.sh"


chmod +x /etc/rc.d/rc.local
```

### 时间操作

#### 1）查看时间。

date （功能描述：显示当前时间）

date +%Y （功能描述：显示当前年份）

date +%m（功能描述：显示当前月份）

date +%d （功能描述：显示当前是哪一天）

date “+%Y-%m-%d %H:%M:%S”（功能描述：显示年月日时分秒）

#### 2）设置时区为中国上海时间（注意不是北京时间）。

cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

#### 3）设置时间。

date -s “yyyy-mm-dd hh:mi:ss”

#### 4）cal 指令

查看日历指令 cal

cal [选项] （功能描述：不加选项，显示本月日历）

案例 1: 显示当前日历 cal

案例 2: 显示 2020 年日历 : cal 2020



### 重启和关机

1）重启

init 6 或 reboot

2）关机

init 0 或 halt

### 清屏

clear

### 查看服务器的ip地址

ip addr

### 查看当前工作目录

pwd

### 改变当前工作目录

cd 目录名

### 列出目录和文件信息

ls -lt 目录或文件名

正则表达式

ls /tmp/exp*.dmp

### 创建目录文件

mkdir 目录名
touch 文件名

### 删除目录和文件

rm -rf 目录或文件列表

```
rm -f .*.*.swp
```

### 移动目录和文件

mv 旧目录或文件名 新目录或文件名

### 复制目录和文件

cp -r 旧目录或文件名 新目录或文件名

### 判断网络是否连通

Windows系统：

ping -n 包的个数 ip地址或域名

Linux系统：

ping -c 包的个数 ip地址或域名

### 用户管理

#### 查询用户信息

id 用户名
whoami/who am I

#### 用户和组相关文件

- /etc/passwd 文件
  用户（user）的配置文件，记录用户的各种信息
  每行的含义：用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录 Shell

- /etc/shadow 文件
  口令的配置文件
  每行的含义：登录名:加密口令:最后一次修改时间:最小时间间隔:最大时间间隔:警告时间:不活动时间:失效时间:标志

- /etc/group 文件
  组(group)的配置文件，记录 Linux 包含的组的信息
  每行含义：组名:口令:组标识号:组内用户列表

#### 增加/删除用户组

1）增加用户组

groupadd 组名

2）删除用户组

groupdel 组名

#### 增加/删除用户

1）增加用户

useradd 用户名
useradd -d 指定目录 用户名
useradd -g 用户组 用户名
passwd 用户名

useradd -n 用户名 -g 组名 -d 用户的主目录

2）删除用户

//保留用户主目录
userdel 用户名
//删除用户以及主目录
userdel -r 用户名

userdel 用户名

#### 修改用户的组

usermod –g 用户组 用户名

#### 修改用户的密码

passwd 用户名

#### 切换用户

su - 用户名

exit/logout

从root用户切换到其它普通用户不需要输入密码

### 修改目录和文件的主人和组

chown [-R] 用户名:组名 目录或文件名列表

-R 选项表示处理各及子目录。

1）把/oracle/home和/oracle/base及其子目录的主人改为oracle，组改为dba。

chown -R oracle:dba /oracle/home /oracle/base

### 查看系统磁盘空间

df [-h] [-T]

-h 以方便阅读的方式显示信息。

-T 列出文件系统类型。

### echo 指令

echo 输出内容到控制台

- 基本语法

echo [选项] [输出内容]

title: 使用rdesktop来在Windows和Linux之间共享数据
date: 2017-02-18 19:33:13
tags:
 - rdesktop
 - RDP
 - Linux
 - Windows
---
### 概述
[rdesktop](http://www.rdesktop.org/)是一个开源的远程桌面客户端，用来从Linux机器连接到Windows机器。它遵循RDP协议（Remote Desktop Protocol），并且操作简洁，功能比较完备。
<!--more-->

### 安装
在Debian发行版上，可以直接用`apt-get`命令安装:
```bash
sudo apt-get install rdesktop
```
别的发行版的安装方式请参看rdesktop项目的GitHub页面:<https://github.com/rdesktop/rdesktop>。

### 连接
最简单的情况，如果你要连接到的Windows机器的IP地址是a.b.c.d, 需要以用户username登录，则可以这样运行rdesktop命令：
```bash
rdesktop -u username a.b.c.d
```
如果你想直接在命令里面使用用户的登录密码，则使用`-p`选项：
```bash
rdesktop -u username a.b.c.d -p my-password
```
如果你想设置登录后的窗口的大小，则采用`-g`选项：
```bash
rdesktop -u username a.b.c.d -p my-password -g 1200x900
```
登录后你会感觉字体显示比较怪，看着很不舒服，可以使用`-x`选项来是字体变得光滑:
```bash
rdesktop -u username a.b.c.d -p my-password -g 1200x900 -x 0x80
```
其中`0x80`还可以改为`0x81`, `0x8F`,分别表示LAN default mode， broadband default mode 和 modem default mode,为不同的"RDP5 experience"。  
以上就是基本的连接选项，也可以通过运行`rdesktop -h`命令来查看所有选项。  

### 共享文件
一个常见的需求是在Windows和Linux系统上共享文件。Samba服务可以解决这个问题，但配置比较复杂。这里我们采用rdesktop来完成这个任务。  
首先在Linux系统下创建一个目录，例如:`/home/username/Pictures`，然后在连接的时候采用`-r disk`选项来进行文件的共享：
```bash
rdesktop -u username a.b.c.d -p my-password -g 1200x900 -x 0x80 -r sound:local -r disk:LinuxPictures=/home/username/Pictures
```
这样在连接到Windows的时候，会在文件资源管理器里面，显示`LinuxPictures`目录。  
这里有两个地方需要注意：
 1. 命令中Linux目录的路径必须采用绝对路径，否则会出错。如上例中，将`/home/username/Pictures`改成`~/Pictures`则会报错。
 2. 为了正常工作，需要同时设置`-r sound:local`，虽然看起来好像没什么关系。关于这个问题的讨论见[这里](http://askubuntu.com/questions/74713/how-can-i-copy-paste-files-via-rdp-in-kubuntu)和[这里](https://github.com/FreeRDP/Remmina/issues/243)。

设置好之后，就可以在Windows和Linux之间通过Pictures目录传输和共享文件了。


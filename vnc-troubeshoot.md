title: VNC使用总结
date: 2017-07-28 20:37:49
tags:
 - Linux
 - Ubuntu
 - VNC
 - Xfce
 - Gnome
 - 总结
---
这是一些使用VNC连接服务器的总结，这些操作都是在Ubuntu操作系统下进行的。  
![](https://lh6.ggpht.com/RcRUeZKNRYaCfoNGMe8Ic8OORBN-_pXgNyNtvNfSQ-5DFl-7CTuTYC2m96BbbV5IQU0=w300)
<!--more-->
### VNC使用 Gnome桌面系统
安装Gnome桌面：
```bash
sudo apt-get install --no-install-recommends ubuntu-desktop gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal     
```
然后修改你的`~/.vnc/xstarup`文件为如下内容：
```bash
#!/bin/sh

# Uncomment the following two lines for normal desktop:
# unset SESSION_MANAGER
# exec /etc/X11/xinit/xinitrc

[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &
x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
x-window-manager &

gnome-panel &
gnome-settings-daemon &
metacity &
nautilus -n &
gnome-terminal &
```
### 使用Xfce桌面
如果要使用xfce桌面的话，通过如下命令来安装：
```bash
sudo apt install xfce4 xfce4-goodies       
```
然后将`~/.vnc/xstartup`改为如下：
```bash
#!/bin/bash
xrdb $HOME/.Xresources
startxfce4 &
```
### 打开gnome-terminal显示“Input/Output Error”
从[这里](https://bbs.archlinux.org/viewtopic.php?id=218510)了解到，可以使用`journalctl -xe`查看报错，我在查看是发现下面的错误信息：
```bash
org.gnome.Terminal[4882]: Non UTF-8 locale (ANSI_X3.4-1968) is not supported!
```
从这里看到，应该是locale设置不对然后导致的错误，即Gnome-Terminal只支持UTF-8的编码。所以这里只需要将locale设置合适即可。有下面两种方法：
#### 1. 修改.bashrc或.zshrc
在.bashrc或.zshrc里面增加如下内容：
```bash
# set LC
LANG=en_US.UTF-8
LANGUAGE=
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC=zh_CN.UTF-8
LC_TIME=zh_CN.UTF-8
LC_COLLATE="en_US.UTF-8"
LC_MONETARY=zh_CN.UTF-8
LC_MESSAGES="en_US.UTF-8"
LC_PAPER=zh_CN.UTF-8
LC_NAME=zh_CN.UTF-8
LC_ADDRESS=zh_CN.UTF-8
LC_TELEPHONE=zh_CN.UTF-8
LC_MEASUREMENT=zh_CN.UTF-8
LC_IDENTIFICATION=zh_CN.UTF-8
LC_ALL=

export LANG
export LANGUAGE=
export LC_CTYPE
export LC_NUMERIC
export LC_TIME
export LC_COLLATE
export LC_MONETARY
export LC_MESSAGES
export LC_PAPER
export LC_NAME
export LC_ADDRESS
export LC_TELEPHONE
export LC_MEASUREMENT
export LC_IDENTIFICATION
export LC_ALL=
```
然后source下，删除之前的VNC会话，重新建立会话，查看是否正确。  

#### 2. 修改系统的locale设置
如果你是管理员的话，可以修改系统的locale设置，使得所有用户都能正确地使用VNC。具体的代码如下：
```bash
sudo update-locale LANG=en_US.UTF-8
sudo update-locale LANGUAGE=
sudo update-locale LC_CTYPE="en_US.UTF-8"
sudo update-locale LC_NUMERIC=zh_CN.UTF-8
sudo update-locale LC_TIME=zh_CN.UTF-8
sudo update-locale LC_COLLATE="en_US.UTF-8"
sudo update-locale LC_MONETARY=zh_CN.UTF-8
sudo update-locale LC_MESSAGES="en_US.UTF-8"
sudo update-locale LC_PAPER=zh_CN.UTF-8
sudo update-locale LC_NAME=zh_CN.UTF-8
sudo update-locale LC_ADDRESS=zh_CN.UTF-8
sudo update-locale LC_TELEPHONE=zh_CN.UTF-8
sudo update-locale LC_MEASUREMENT=zh_CN.UTF-8
sudo update-locale LC_IDENTIFICATION=zh_CN.UTF-8
sudo update-locale LC_ALL=
```
这样操作后同样需要重新建立VNC会话。  

### VNC连过去后，命令行字体挤在一起，看不清楚
这个原因也是因为locale设置的不对，设置了中文字体导致的问题，所以同样地，按照上面所说的更改locale的方法，更新locale即可。 

### VNC中，按Tab不自动补全，而是跳转到别的Terminal窗口
这个问题的解决方法是实验室师兄提供的，打开`~/.config/xfce4/xfce-perchaannel-xml/xfce4-keyboard-shortcuts.xml`文件，将其中的
```xml
<property name="&lt;Super&gt;Tab" type="string" value="switch_window_key"/>
```
修改为
```xml
<property name="&lt;Super&gt;Tab" type="empty"/>
```
然后用`vncserver -kill :portid` kill掉连接，再用`vncserver -geometry 1920x1080 :portid`来新建连接。  

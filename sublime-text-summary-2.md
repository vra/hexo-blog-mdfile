title: Sublime Text 使用技巧2
id: 507
categories:
  - Sublime Text
date: 2015-05-29 17:33:14
tags:
 - Sublime Text
---

## 1\. 安装包管理工具Package Control

包管理工具是安装插件的一个简单有效的方法，安装完Package Control后，就可以用**Ctrl-Shift-P** 快捷键来安装插件了。
包管理器的安装方式:用_Ctrl-`_快捷键打开命令行，然后在命令行中输入如下代码：

```python
import urllib2,os,hashlib; h = 'eb2297e1a458f27d836c04bb0cbaf282' + 'd0e7a3098092775ccb37ca9d6b2e4b7d'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler()) ); by = urllib2.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); open( os.path.join( ipp, pf), 'wb' ).write(by) if dh == h else None; print('Error validating download (got %s instead of %s), please try manual install' % (dh, h) if dh != h else 'Please restart Sublime Text to finish installation')
```
然后按回车，之后重启Sublime Text 2，如果在**Preferences-&gt;Package Setttings**菜单里出现**Package Control**，就说明安装成功了。如果使用的是Sublime Text 3，可以看着[这个链接](https://packagecontrol.io/installation)。

<!--more-->

## 2\. 安装插件Terminal

这个插件用来打开一个命令终端，而且这个命令终端的路径就是当前编辑文件或项目所在路径，所以这条命令非常实用，可以在Sublime Text 2里面编辑好文件后，立即在命令行里面编译什么的，很方便。
安装方法：用**Ctrl-Shift-P**打开窗口，输入**Package Control Install**，按回车，再输入**Terminal**，回车之后就开始安装，可以通过左下角的小字查看进度。


## 3\. 安装主题Theme

在包安装界面，输入**Theme**，即可看到所有的主题，选择自己喜欢的下载。下载后，修改**Preferences-&gt;Settings-User**，在打开的文件中加入下面一行:

```bash
"theme": "Nexus.sublime-theme",
```

保存配置文件，主题立即改变。


## 4\. SublimeLinter：代码检查插件

SublimeLinter据说是一个很好用的代码检查插件，但没用过，所以就只是记录下……

    
## 5\. SFTP插件

sftp是一个在Sublime Text 2里面可以直接登陆sftp和ftp账号的插件，登陆还可以浏览、修改账号上的内容，有了sftp，就再也不需要FileZilla了~
**突然惊喜地发现，SFTP插件也支持SSH，所以以后freeshell可以比较随意地登了。**

 1.  安装：插件安装，搜索sftp，安装即可。
 2.  安装后，按**Ctrl-Shift-P**，输入sftp，可以看到有`Browser Server`，`Delete Server`，`Edit Server`，`Setup Server`等命令，首先选择**Setup Server**来增加第一个连接。修改默认的配置文件并保存，即为一个连接。配置文件主要有如下内容：

 ```bash
 // sftp, ftp or ftps
 "type": "sftp",

 "host": "ssh.freeshell.ustc.edu.cn",
 "user": "user",
 "password": "password",
 "port": "88888",

 "remote_path": "/",
 ```

 其中

  1.  `type`表示连接的会话协议类型，**注意ssh设置成sftp即可进行连接**
  2.  `host`是要连接的主机名
  3.  `user`是要进行连接的用户名
  4.  `password`是用户的密码
  5.  `port`是进行连接的端口，ftp默认是22端口


## 6\. 论文还没改完，去写论文了，下次再写~

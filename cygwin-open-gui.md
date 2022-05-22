---
title: Cygwin环境下查看远程服务器的图片
date: 2020-03-01 09:32:12
tags:
 - Cygwin
 - Linux
---
Linux系统下，`SSH -X`能够将 ssh 连接到的远程服务器上的图形化显示转发到本地，因此可以方便地查看服务器上的结果。不过在Cygwin下，使用`ssh -X`选项并不work。

调查后发现是需要用Cygwin的安装文件安装`xorg-server` 和 `xinit`这两个包，然后在一个终端执行`startxwin`，一直开着，在别的终端进行ssh连接与显示，发现就可以了。

另外发现Cygwin的一个缺点是，每次安装包都得重新运行一遍安装文件，还是没有`apt`来得方便，所以后面试试能否用WSL替代Cygwin来工作。


## 参考
1. <https://unix.stackexchange.com/a/227937>
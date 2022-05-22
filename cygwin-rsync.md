---
title: Cygwin 使用rsync 报错解决
date: 2020-03-01 09:32:12
tags:
 - Cygwin
 - Linux
---
在 Cygwin 下使用`rsync`时，报下面的错误：
```bash
rsync: connection unexpectedly closed (0 bytes received so far) [receiver] 
rsync error: error in rsync protocol data stream (code 12) at io.c(600) [receiver=3.0.5]
rsync: connection unexpectedly closed (0 bytes received so far) [sender]
rsync error: error in rsync protocol data stream (code 12) at io.c(610) [sender=3.0.8]
```
由于`rsync`是通过`ssh`工具来传数据的，通过`which ssh` 查看，发现使用的是 Windows 自带的 SSH，所以报错，因此用 Cygwin 的安装程序重新安装`openssh`包，再打开终端，默认使用的`ssh`就变成 Cygwin 下的了，此时使用`rsync`命令就不再报错了。
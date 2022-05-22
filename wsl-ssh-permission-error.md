---
title: Windows Subsystem for Linux 下 SSH permission error 解决
date: 2020-03-04 20:26:22
tags:
 - Windows
 - Linux
 - WSL
 - SSH
 - Trick
---
在 Windows SubSystem for Linux (WSL) 下，使用 `ssh` 命令的时候报下面的错：
```bash
Bad owner or permissions on /home/yunfeng/.ssh/config
```
搜索了一下，发现修改下 `config` 文件的权限就可以了：
```bash
chmod 600 ~/.ssh/config
```

参考
1. <https://github.com/Microsoft/WSL/issues/483>

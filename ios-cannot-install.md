---
title: Xcode无法安装ios程序的一种情况记录
date: 2022-04-06 14:06:46
tags:
- iOS
- Mac
---
在用Xcode调试ios代码的时候，发现代码可以正常编译，但是安装到手机的时候，提示"App installation failed: Could not write to the device"。在网上找了很多回答，都没能解决。后来发现原因是在要复制到ios的目录中添加了一个软链接，导致出错。删除软链接后，安装正常。这应该是一个比较少见的原因，记录一下。

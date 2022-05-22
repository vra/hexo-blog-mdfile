---
title: 你不应该知道的知识之如何在Ubuntu 16.04上安装Pip2
date: 2019-09-07 22:27:02
tags:
 - Python
 - Pip
 - Ubuntu
---
## 概述
虽然2020年官方就不支持Python2系列了，不过有时你还是会用到Python2，这时候为了安装某个包(如numpy)，你需要`pip2`。而`pip2`一般比较新的系统是没带的。下面记录如何安装它。

<!--more-->

## 命令
很简单，两行命令搞定：
```bash
wget https://bootstrap.pypa.io/get-pip.py
sudo python2.7 get-pip.py
```
使用时输入`pip2`即可：
```bash
pip2 -V
```


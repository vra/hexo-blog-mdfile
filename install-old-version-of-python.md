---
title: 你不应该知道的知识之如何安装老版本的Python
date: 2019-09-07 22:12:13
tags:
 - Python
 - Ubuntu
 - Linux
---
## 概述
由于某些奇怪的原因（如项目中要用某个用Python3.4编译的库），你可能需要安装官方停止支持的Python版本（如Python2.5, Python2.6, Python3.3, Python 3.4或者更老的版本），
直接通过`sudo apt install python3.4`是没法安装的，因为Ubuntu 16.04移除了对Python3.4的支持。  
作为不应该知道的知识的一部分，这里详细记录下在Ubuntu 16.04下安装旧版本的Python的方式，如果在2029年，因项目你需要安装Python3.4，或许本文可以帮到你。

<!--more-->

## 具体步骤
为了使用`add-apt-repository`，需要先安装下面的包：
```bash
sudo apt-get install software-properties-common

```
 1. 增加`deadsnakes` PPA (名字好评)
 ```bash
 sudo add-apt-repository ppa:deadsnakes/ppa
 ```
 2. 使用`apt`安装pythonx.y:
 ```bash
 sudo apt install python3.4
 ```
 如果需要安装`libpython3.4`，可以使用类似下面的命令：
 ```bash
 sudo apt install libpython3.4 libpython3.4-dev libpython3.4-minimal
 ```

---
title: Ubuntu上安装Python3.7
date: 2019-05-03 20:20:45
tags:
 - 总结
 - Python
 - Ubuntu
 - Linux
---
## 概述
在有些情况下，如安装某个比较Cool的工具的时候，需要用到Python3.6+。这时候，可以选择从[Python官网](https://www.python.org/)下载源代码，然后编译。不过编译可能会因为各种各样的问题而出错。对于只是想**安装**高版本的Python以便来使用Cool的工具的我来说，
从头一步步地解决这些编译问题，并不是我想要的，因此能不能有一种直接`apt install`来安装Python的途径呢？答案是Yes，下面详述（也就三条命令）。

<!--more-->

为了使用`add-apt-repository`，需要先安装下面的包：
```bash
sudo apt-get install software-properties-common
```

在Ubuntu的[包管理网站](https://launchpad.net)上，有一个专门的PPA，维护了从Python2.3到Python3.8的二进制包，可以直接增加PPA源后安装Python:
```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install -y python3.7
```
如果想要安装别的版本的Python，将上面的`3.7`改成对应的版本号即可：
```bash
sudo apt install -y python3.6
```

安装好对应的版本后，可以配合[Pipenv](https://docs.pipenv.org/en/latest/)一起使用，实现多版本Python的管理和包安装。

## 参考
1. <https://askubuntu.com/questions/865554/how-do-i-install-python-3-6-using-apt-get>
2. <https://launchpad.net/~deadsnakes/+archive/ubuntu/ppa>

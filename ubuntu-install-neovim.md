---
title: Ubuntu安装NeoVim:一种最简单的方法
date: 2019-03-13 13:36:51
tags:
 - Linux
 - Ubuntu
 - NeoVim
 - Vim
 - 总结
---
[NeoVim](https://neovim.io)是Vim的一个拓展版本，用起来比Vim爽一些。下面简要记录下在Ubuntu 16.04上安装NeoVim的过程，其实比较简单。
<!--more-->

为了使用`add-apt-repository`，需要先安装下面的包：
```bash
sudo apt-get install software-properties-common
```
然后选择stable或者unstable版本进行安装，见下。
## 安装stable版本, version=0.2.2
```bash
sudo add-apt-repository ppa:neovim-ppa/stable
sudo apt update
sudo apt install -y neovim
```

## 安装unstable版本, version=0.4.0-dev
因为某些插件只支持`0.3`及以上的版本，因此为了使用插件需要安装unstable版本：
```bash
sudo add-apt-repository ppa:neovim-ppa/unstable
sudo apt update
sudo apt install -y neovim
```

安装后就可以使用了，用命令`nvim`即可打开Neovim，建议继续阅读[vim-plug](http://vra.github.io/2019/03/12/vim-plug-intro/)来了解NeoVim的插件安装工具。

## 参考
1. <https://launchpad.net/~neovim-ppa/+archive/ubuntu/stable>
2. <https://launchpad.net/~neovim-ppa/+archive/ubuntu/unstable>

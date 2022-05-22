---
title: 从源代码编译安装tmux
date: 2019-10-11 17:16:13
tags:
 - tmux
 - Linux
---

## 概述
为了使用新版tmux的特性，需要在Ubuntu 16.04上安装高版本的tmux，没有找到现成的ppa，因此搜到了一个从源代码安装的脚本，这里记录下来。
<!--more-->

## 安装
tmux的源代码在GitHub上，地址是 <https://github.com/tmux/tmux>，可以在Release页面下载源代码然后进行编译，已编译tmux 2.9为例，具体操作如下：

```bash
sudo apt-get update
sudo apt-get install -y libevent-dev libncurses-dev make automake
cd /tmp
wget https://github.com/tmux/tmux/archive/2.9.tar.gz
tar xvzf 2.9.tar.gz
cd tmux-2.9/
bash autogen.sh
./configure && make
sudo make install
cd ..
rm -rf ./tmux-2.9*
```

Done，在命令行查看tmux版本，如果是2.9就说明okay了。**另外记得关掉所有的tmux session，重新打开后环境的修改才会生效。
```bash
tmux -V
tmux 2.9
```

## 参考：
1. <https://gist.github.com/japrescott/aa15cb024fe38ea36849f5f62c3314a3>

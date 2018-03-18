title: Ubuntu命令行显示中文
date: 2018-01-13 18:32:18
tags:
 - Linux
 - Ubuntu
---
我们的Ubuntu服务器用命令行显示中文一直有问题，网上找资料说安装zhcon，依旧解决不了我们的问题。因此这里调查了下可能的原因，将其记录下来。
<!--more-->

### 解决办法
将如下的设置项放到`~/.bashrc`中，然后执行`source ~/.bashrc`:
```bash
LANG=en_US.UTF-8
LANGUAGE=
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC=zh_CN.UTF-8
LC_TIME=zh_CN.UTF-8
LC_COLLATE="en_US.UTF-8"
LC_MONETARY=zh_CN.UTF-8
LC_MESSAGES="en_US.UTF-8"
LC_PAPER=zh_CN.UTF-8
LC_NAME=zh_CN.UTF-8
LC_ADDRESS=zh_CN.UTF-8
LC_TELEPHONE=zh_CN.UTF-8
LC_MEASUREMENT=zh_CN.UTF-8
LC_IDENTIFICATION=zh_CN.UTF-8
LC_ALL=
```
其中最重要的是最后一项设置`LC_ALL`，因为其默认设置是`LC_ALL=C`，`C`代表覆盖掉 LANG 和所有 LC_* 变量的值, 将其设置为系统默认值。因此该项设置为空即可采用自定义设置。


### 参考内容
1. <https://wiki.archlinux.org/index.php/Locale_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>

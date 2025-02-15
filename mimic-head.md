---
title: mimic-head-实时摄像头驱动图片动起来
date: 2024-07-13 08:08:12
tags:
  - AI
  - Deep Learning
  - LivePortrait
  - 快手
  - Kwai
  - Python
  - pip 
---
整了一个快手人头驱动项目[LivePortrait](https://github.com/KwaiVGI/LivePortrait)的demo，一键安装（自动下载模型），同时增加了官方demo中没有的实时摄像头驱动，也支持cpu和mps这两个后端了。
<!--more-->

安装超easy:

```bash
pip install mimic_head
```

使用超easy:

```bash
mimic_head run
```
打开浏览器访问127.0.0.1:7860就可以开始玩了。

摄像头驱动效果在[这里](https://zhuanlan.zhihu.com/p/708618764)


不得不说，快手这个效果真的牛，太好玩了。

源码：https://github.com/vra/mimic_head

欢迎star，fork and 魔改。

Have fun!

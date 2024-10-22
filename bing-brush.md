---
title: Bing Brush-Python代码和命令行中调用必应 DALL·E 3文生图模型
date: 2023-11-04 23:22:03
tags:
 - AI
 - Python
 - pip
 - DALL·E
 - Bing
---

###  1. 说明

今早看到一个好玩的项目，利用Bing Image Creator 来生成每日诗词的图像，研究了一下，发现有人提供了[BingImageCreator](https://github.com/acheong08/BingImageCreator)仓库来调用Bing的API在代码中生成图像，但还需要下载源码，没有提供cli，cookie怎么获取也没有讲太细。

因此我基于这个仓库，做了一些精简和封装，提供了一个可以直接pip安装的工具[bing_brush](https://github.com/vra/bing_brush), 获取cookie后可以直接命令行调用。
<!--more-->
  
![](https://pic1.zhimg.com/80/v2-4d60e7c55a9388e56903c58fd3b1432f_1440w.png?source=d16d100b)

整体流程很简单：
```bash
pip install bing_brush
# 获取bing.com的cookie，见下文
bing_brush -c cookie.txt -p 'a cute panda eating bamboos' -o output_folder
```


就会output_folder 下生成4张图像：

![](https://pica.zhimg.com/80/v2-9f13f504f3431d1018acd2bf8d3a7241_1440w.png?source=d16d100b)

源码：[vra/bing_brush (github.com)](https://github.com/vra/bing_brush)
欢迎Watch, Star, Fork 和Contribute！

### 2. cookie获取

整个过程中稍微有些繁琐的是获取cookie，详细操作见下。

首先打开 [https://www.bing.com/images/create](https://www.bing.com/images/create)

如果访问不了的话，那这个工具也没法使用，因此确保这个页面可以正常打开。

![](https://picx.zhimg.com/80/v2-aa9f3e5f8e645d02f9ad174fa11a0f50_1440w.jpeg?source=d16d100b)

然后按F12，打开开发者页面，然后刷新页面，会看到很多请求，选择任一类型为xhr的请求，点击前面的lianjie：

![](https://picx.zhimg.com/80/v2-173983dcdd28069d41dce7af3f2d61eb_1440w.jpeg?source=d16d100b)


进入详情页面后，往下翻找到Cookie 部分，将对应的右边的复制到cookie.txt即可，后面-c 指定这个路径就行。

![](https://picx.zhimg.com/80/v2-cd8f0e48096c0b990a931764610bf5ad_1440w.jpeg?source=d16d100b)
  

###  3. 使用流程

pip安装bing_brush，并且获取cookie后，就可以用一条命令来运行图像生成：

bing_brush -c cookie.txt -p 'a cute panda eating bamboos' -o output_folder

然后就可以发挥你的创意来在命令行跑图了。

### 4. Python代码中使用

pip 安装后，也可以在Python代码中使用 Bing Brush:

from bing_brush import BingBrush

brush = BingBrush(cookie='cookie.txt')
brush.process(prompt='a cute panda eating bamboos', out_folder='output_folder')

### 5. 彩蛋

这个项目的Logo也是用Bing生成的，prompt如下：

> A minimalist logo vector image, square-shaped, with a magical brush implemented in Python language in the center, colorful, digital art

画出了三张logo，最后选择第三张作为项目的Logo

![](https://picx.zhimg.com/80/v2-28772285cca47864cbd9a6bf396a6bb1_1440w.png?source=d16d100b)

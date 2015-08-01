title: OpenCV配置——在Linux中使用OpenCV
id: 481
categories:
  - 学习总结
  - 计算机视觉
  - OpenCV
date: 2015-04-25 10:37:40
tags:
---

这篇博客介绍在Linux中的gcc和g++编译环境下如何使用cmake来编译OpenCV源代码。我基本是按照OpenCV官方的[说明文档](http://docs.opencv.org/doc/tutorials/introduction/linux_install/linux_install.html)，一步步地进行的，所以表述不清楚的地方还请参照原文。

<!--more-->

## 1\. 编译环境

*   操作系统：Ubuntu 14.10
*   gcc 版本: 4.9.1
*   cmake 版本： 2.8.12.2
*   opencv版本： 2.4.10

## 2\. 依赖包安装

依赖包包括在编译的时候要用到一些软件，像gcc，cmake；还有一些是下载opencv需要的工具，像Git；还有一些编译opencv所必需的，像ffmpeg 或libav ；还有一些是可选的包等等。可以通过下面几条命令来安装这些依赖包：

```bash
sudo apt-get install build-essential 
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
```

运行完这三条命令，依赖包就安装好了！


## 3\. 获取OpenCV源代码

官方网站上给了2种获取源代码的方式：

 1.  从[Sourceforge](http://sourceforge.net/projects/opencvlibrary/)上获取最新的稳定版(lastest staable)的OpenCV，下载完解压即可。
 2.  从[github](http://github.com/itseez/opencv)上下载最前沿的版本。也可以在命令行下载：
 ```bash
    git clone https://github.com/Itseez/opencv.git
 ```

## 4\. 用cmake编译OpenCV

下载完源代码后，就可以用cmake来编译OpenCV了。
解压下载得到的opencv包，然后进入包目录，在下面进行操作。

 1.  创建release目录，然后将进入该目录，下面编译都是针对Release版来进行编译的：

 ```bash
 mkdir release
 cd ~/release
 ```

 2.  执行cmake命令：

 ```bash
 cmake -D CMAKE_BUILD_TYPE =RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local
 ```    
 上面的`CMAKE_BUILD_TYPE =RELEASE`指明编译的版本是Release版，`CMAKE_INSTALL_PREFIX=/usr/local`指明编译后的可执行程序的存放目录。

 3.  执行make和install：

 ```bash
 make    
 sudo make install
 ```

 如果没有出错的话，OpenCV的整个编译过程就完成了！ 如果有错误，那就复制错误内容，到网上查找解决办法，一般来说这是个很痛苦的过程，所以希望你有好运气，一次编译就能过：)

## 5\. 在gcc/g++编译时使用opencv

在g++里面编译使用了opencv库的程序时，只需要在后面添加``pkg-config opencv --cflags --libs``即可，如下例子：

```bash
g++ -o main main.cpp`pkg-config opencv --cflags --libs`
```

以上就是Linux环境下使用OpenCV的一些总结。

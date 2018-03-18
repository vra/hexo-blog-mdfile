title: Linux下使用自定义路径来运行OpenCV
date: 2017-12-04 15:50:57
tags:
 - Linux
 - OpenCV
 - Caffe
---
有的时候系统安装的OpenCV版本和你需要的版本不一样，而你又没有权限或者为了兼容不能修改系统的OpenCV，这个时候你就得自己编译OpenCV，然后在需要的代码里面引用你编译的版本。整个过程不复杂，但是之前一直没搞清楚，最近经师弟点拨才明白，这里记录一下。  
<!--more-->
我之前写过一篇在Linux下编译OpenCV的[博客](https://vra.github.io/2015/04/25/opencv-linux-install/)，大家可以参考下，我这里只记录与其中不同的部分。  
### 修改`CMAKE_INSTALL_PREFIX`
默认的`CMAKE_INSTALL_PREFIX`为`/usr/local`，而我们不想安装到这里，所以这里修改其为你想要保存的目录，如`/home/username/local`:
```bash
cmake -D CMAKE_INSTALL_PREFIX=/home/username/local ..
```
另外一个小问题，如果你在cmake的时候出现下面信息：
```bash
ICV: Downloading ippicv_linux_20151201.tgz...
CMake Error at 3rdparty/ippicv/downloader.cmake:73 (file):
  file DOWNLOAD HASH mismatch

    for file: [/home/pauka/opencv/3rdparty/ippicv/downloads/linux-808b791a6eac9ed78d32a7666804320e/ippicv_linux_20151201.tgz]
      expected hash: [808b791a6eac9ed78d32a7666804320e]
        actual hash: [f166287239920c4a16e6f8870e15ef79]
```
即ippicv这个包下载不了，你可以在cmake里面加`-D WITH_IPP=OFF`来禁用这个包，也可以手动下载，下载方式见[这里](https://github.com/opencv/opencv/issues/5973)。  

cmake完后，继续执行`make`和`make install`。注意这里`make install`前面不需要`sudo`，因为我们不修改系统目录，不需要管理员权限。  

### 修改lib和include，增加OpenCV的目录
为了在编译的时候找到我们的OpenCV，需要修改lib和include路径，把OpenCV的目录加到里面去。例如编译Caffe的时候，修改`INCLUDE_DIRS`和`LIBRARY_DIRS`，将OpenCV的目录加进去。加入我们的OpenCV的编译后存放路径是`/home/username/local/`,那么对应的lib和include目录应该是`/home/username/local/lib`和`/home/username/local/include`。  

### 修改`PKG_CONFIG_PATH`环境变量
这个环境变量是给`pkg-config`这个工具增加额外的查找目录的，pkg-config会默认查找`/usr/lib/pkgconfig`和`/usr/share/pkgconfig`下的`.pc`配置文件，额外的目录通过设置`PKG_CONFIG_PATH`来增加。我们这里将自己的OpenCV放进去，即可：
```bash
export PKG_CONFIG_PATH=/home/username/local:$PKG_CONFIG_PATH
```

### 检查设置是否正确
如何验证编译别的库的时候找到的是我们编译的OpenCV而不是系统的呢？可以通过`pkg-config`命令来确定：
```bash
pkg-config --modversion opencv
```
如果版本是你编译的版本，那就说明找到了，可以正常用了。  

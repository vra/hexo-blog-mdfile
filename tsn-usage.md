title: TSN Usage——如何编译和使用temporal-segment-networks
date: 2017-03-31 09:47:23
tags:
 - Action Recognition
 - CNN
 - TSN
 - GitHub
 - CUDA
 - Caffe
 - OpenCV
---
[TSN](https://github.com/yjxiong/temporal-segment-networks)是"temporal-segment-networks"的简称，是视频动作识别任务里面当前最好的方法。虽然这个结构是在ECCV2016的论文里面提出来的，代码也放出来挺长时间了，但是这个项目里面集合了Caffe， OpenCV，CUDA，CUDNN等几大神坑项目，不同版本之间的依赖、选择等问题很麻烦，因此我之前编译了好几次都没有能够编译成功。这次花了近一天的时间来重新编译了一下整个项目，虽然还是有些问题，例如MPI编译没有通过，CUDA8貌似不支持，CuDNN v5好像也不支持，但最后总算是编译通过，可以运行了。所以记录一下整个的过程，期望对自己和别人能够有所帮助。
<!--more-->

### 1. OpenCV编译
因为TSN这个项目里面有提取光流的code，其中利用的是OpenCV里面的利用GPU来提取光流的代码，所以需要用到OpenCV。虽然可以使用系统已经编译好的，但是在编译dense_flow的时候发现还依赖[opencv_contrib](https://github.com/opencv/opencv_contrib)中的库，所以为了避免重新编译系统的OpenCV影响别的用户，我自己编译了一个新的版本的OpenCV，放在自己的目录下。  
因为我们服务器上已经装过了3.1.0版本的OpenCV(可以通过`pkg-config --modversion opencv`命令来查看OpenCV的版本)，所以为了避免编译时寻找include目录中的文件的时候报错，这里在自己的目录下安装3.1.0版本的OpenCV和外部的库（因为dense_flow代码需要用到额外的库）。

#### 1. 下载最新版本的OpenCV 和 opencv_contrib, 确保两个库都切换到3.1.0这个分支: 
```bash
cd /data5/yunfeng/Dev/git
git clone https://github.com/opencv/opencv.git
cd opencv
git checkout 3.1.0
cd ../
git clone https://github.com/opencv/opencv_contrib.git
cd opencv_contrib
git checkout 3.1.0
```
##### 2. 利用cmake和make命令来编译OpenCV
```bash
cd ../opencv
mkdir release; cd release
cmake -D OPENCV_EXTRA_MODULES_PATH=/data5/yunfeng/Dev/opencv_contrib/modules -D CMAKE_BUILD_TYPE=RELEASE -D WITH_TBB=ON -D BUILD_NEW_PYTHON_SUPPORT=ON -D INSTALL_C_EXAMPLES=ON -D INSTALL_PYTHON_EXAMPLES=ON -D BUILD_EXAMPLES=ON -D ENABLE_FAST_MATH=1 -D CUDA_FAST_MATH=1 -D WITH_CUBLAS=1 -D CMAKE_INSTALL_PREFIX=/data5/yunfeng/local -D CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-7.5 ..
make -j40
make install -j40
```
这里有几个地方需要注意:
 * 编译OpenCV的同时编译opencv_contrib，需要在cmake里面设置`-D OPENCV_EXTRA_MODULES_PATH=/path/to/opencv_contrib/modules`, 其中参数的值就是到opencv_contrib仓库的modules子目录的绝对路径
 * 通过设置`-D CUDA_FAST_MATH=1` 和 `-D WITH_CUBLAS=1`来启用CUDA和CUBLAS
 * 通过设置`-D CMAKE_INSTALL_PREFIX=/path/to/local`来设置编译生成的lib和bin目录存放的位置，如果不设置的话默认是在`/usr/local`目录里面，因此需要管理员权限才能执行最后的`make install`
 * 通过设置`-D CUDA_TOOLKIT_ROOT_DIR=/path/to/cuda`来设置CUDA的路径，不设置话cmake会在系统目录里面寻找，如果系统有多个CUDA版本的话，找到的可能不是我们需要的，所以还是显式地写到参数列表里比较稳妥
 * 此外还可以设置`-D CUDNN_INCLUDE="/path/to/cuda/include"` 和`-D CUDNN_LIBRARY="/path/to/cuda/lib64"` 来设置CuDNN的路径

cmake的时候报错：
```bash
-- ICV: Downloading ippicv_linux_20151201.tgz...
CMake Error at 3rdparty/ippicv/downloader.cmake:73 (file):
  file DOWNLOAD HASH mismatch

    for file: [/path/to/opencv/3rdparty/ippicv/downloads/linux-808b791a6eac9ed78d32a7666804320e/ippicv_linux_20151201.tgz]
      expected hash: [808b791a6eac9ed78d32a7666804320e]
        actual hash: [f166287239920c4a16e6f8870e15ef79]
```
这个问题可以通过直接在[这里](https://github.com/Itseez/opencv_3rdparty/tree/ippicv/master_20151201/ippicv)下载ippicv_linux_20151201.tgz 文件，并放置到`opencv/3rdparty/ippicv/downloads/linux-808b791a6eac9ed78d32a7666804320e/`目录下，然后重新运行cmake即可。  


### 2. 编译TSN
TSN代码里面包含3个submodule，分别是opencv2.4.12, 提取光流的dense_flow和修改过的caffe caffe-action。我们这里已经编译过OpenCV，所以接下来编译dense_flow和caffe-action, 作者给出的`build_all.sh`执行会有些问题，我们可以参照这个文件的内容，自己一步步地编译各个部分。  
#### 1. 编译dense_flow
dense_flow依赖于libzip-dev这个包，可以通过系统的包管理器安装。我本来是想自己编译的，但是编译后，在make dense_flow 的时候还是报错，最后还是让管理员老师装了这个包。 
装完依赖后，开始执行cmake，使用`OpenCV_DIR`参数来设置OpenCV目录，指向我们自己刚才编译的OpenCV。
```bash
cd /data5/yunfeng/Dev/git
git clone --recursive https://github.com/yjxiong/temporal-segment-networks tsn
cd tsn
cd lib/dense_flow
mkdir build; cd build
OpenCV_DIR=/data5/yunfeng/Dev/opencv/release cmake -D CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-7.5 ..
make -j40
make install -j40
```
如果`libzip-dev`这个包是自己安装的，且没有放在系统目录下，你需要增加几个选项来执行cmake，像下面这样：
```bash
OpenCV_DIR=/data10/yunfeng/Dev/git/opencv/release cmake -D CUDA_TOOLKIT_ROOT_DIR=/data1/yunfeng/cuda -D LIBZIP_LIBRARY=/data1/yunfeng/local/lib -D LIBZIP_INCLUDE_DIR_ZIP=/data1/yunfeng/local/include -D LIBZIP_INCLUDE_DIR_ZIPCONF=/data1/yunfeng/local/lib/libzip/include ..
```
#### 2. 编译caffe-action
作者原来的代码是通过MPI来并行运行的，所以需要通过如下的cmake命令来编译caffe:
```bash
OpenCV_DIR=/data5/yunfeng/Dev/opencv/release cmake -DUSE_MPI=ON -D CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-7.5 -DMPI_CXX_COMPILER="/usr/bin/mpicxx" ..
make -j40
```
但是这样编译的时候，CuDNN会报错，没有办法，我就用常规的make来编译了caffe-action:
```bash
cd lib/caffe-action
cp Makefile.config.example Makefile.config
vim Makefile.config # change the settings
make -j40
```
这样整个工程就编译完了，好像也不是很复杂，但是出现问题解决不了的时候还是挺令人烦躁的。

### 3. Troubleshoot
这里列出来一些编译和使用这个项目过程中常出现的问题，大多和OpenCV, Caffe, CUDA和CuDNN相关
1. 在使用OpenCV的CommandLineParser的时候，总提示" FATAL [default] Check failed: [video_stream.isOpened()] Cannot open video stream "true" for optical flow extraction.",这是因为作者原来的代码`tsn/tools/build_of.py`文件里面，使用OpenCV的CommandLineParser的时候，参数是以空格分隔的，如`-f {} -x {} -y {} -i {} -b 20 -t 1 -d {} -s 1 -o {} -w {} -h {}`, 但是新版的OpenCV里面都是用等号`=`来分割的，不知道是不是某个版本修改了接口还是怎么回事，将空格改成等号即可，如`-f={} -x={} -y={} -i={} -b=20 -t=1 -d={} -s=1 -o={} -w={} -h={}`。  
2. 在使用CUDA8来编译项目的时候，会提示"libnppi.so: undefined reference to `nppGetMaxThreadsPerSM", 网上的回答也很少，可能是CUDA8出来没多久，讨论还不是很多吧。所以我暂时认为这个项目不支持CUDA8。  
3. 使用OpenCV2.4.12编译的时候，会提示"/usr/include/opencv2/nonfree/features2d.hpp:81:5: error: ‘AlgorithmInfo’ does not name a type AlgorithmInfo* info() const;"。这是因为我们服务器上已经装了OpenCV3.1.0，所以在编译的时候，会找系统目录下的头文件，而3版本的头文件和2版本的头文件不一致，导致出现这个问题。按理来说，这个问题可以通过修改头文件寻找路径，使得编译器使用2版本的头文件即可，但是我不知道怎么在cmake的时候指定头文件。。所以没办法，还是采用了3版本的OpenCV来编译。有关的讨论见<https://github.com/arrenglover/openfabmap/issues/14>。



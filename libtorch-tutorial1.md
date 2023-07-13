---
title: Libtorch系列教程1：一个丝滑的C++ Tensor库
date: 2023-02-25 03:03:51
tags:
- Pytorch
- Libtorch
- C++
- Python
---
系列教程列表：
+ [Libtorch系列教程1：一个丝滑的C++ Tensor库](https://vra.github.io/2023/02/25/libtorch-tutorial1/) 
+ [Libtorch系列教程2：torch::Tensor的使用](https://vra.github.io/2023/02/25/libtorch-tutorial2/) 

## 1. 概述
[Libtorch](https://pytorch.org/cppdocs/installing.html)是Pytorch的C++接口，实现了在C++中进行网络训练、网络推理的功能。

除此之外，由于Libtorch中的大部份接口都是与Pytorch一致的，所以Libtorch还是一个很强大的张量库，有着类似Pytorch的清晰接口，这在C++中很难得的。如果你用过C++ Tensor库，就会发现写法比较复杂，学习成本。因为强类型的限制和通用容器类型的缺失，C++相比Python天然更复杂，库设计者因为语言使用习惯，以及为了性能等因素，设计的接口一般都是高效但难用的。而Libtorch采用了与Pytorch类似的函数接口，如果你使用过Pytorch的话，使用Libtorch学习成本很低，后面会看到具体的例子。

另一个问题是，很多Python库中基础的操作，例如`numpy.einsum`函数，在C++中没有合适的替代，看看[这些](https://stackoverflow.com/questions/65347170/numpy-einsum-equivalent-for-xtensor-c)搜索你就知道了。Libtorch解决了这个问题，Pytorch中有的它都有，所以在C++中可以简单地用`torch::einsum`来使用einsum函数，简直是C++开发者的福音。

此外Libtorch 是支持GPU的，主要用于模型的推理过程，但我猜测使用GPU的话，Libtorch的Tensor操作在速度上相比别的C++ Tensor 库可能有优势，具体速度需要测试对比。当然使用C++代码的话速度不是瓶颈，本身CPU代码就够快了。

Libtorch另一个优势是编译简单，只要你安装了Pytorch，Libtorch就可以直接使用，省去了复杂的安装和配置，一分钟内就能跑起来一个简单的的示例程序。

总结来说，Libtorch有以下很吸引人的特性：
+ 强大如Numpy和Pytorch的C++ Tensor库，写法优雅丝滑，并且是支持GPU的。
+ 可以训练神经网络
+ 可以推理神经网络模型，用在C++环境的模型部署场景
+ 编译简单

由于Pytorch开发团队是以Python优先的思路来进行Pytorch的开发的，因此我感觉Libtorch的重视程度不是很高，文档和教程也比较少，官网的示例也几乎没有，因此写一个比较完善的教程是比较有意义的。

这个系列文章中，我会对Libtorch 的Tensor库和推理神经网络过程进行介绍，因为这些内容在实际对于用Libtorch来进行网络训练的部分进行跳过，因为这部分使用的场景不是很多（用Python训练网络比C++香多了)。

本篇以Mac下的操作为例，对Libtorch的安装和简单使用进行介绍，后续内容近期会更新，敬请关注。
<!--more-->

## 2. Libtorch 安装
如果你已经安装过Pytorch，那么就不用额外安装Libtorch了，因为Pytorch自带了Libtorch的CMake config 文件，使用`torch.utils.cmake_prefix_path`语句就能打印出来，可以直接被CMake使用，编译时添加如下的选项：
```bash
-DCMAKE_PREFIX_PATH=`python -c 'import torch;print(torch.utils.cmake_prefix_path)'
```

如果没有安装过Pytorch，那直接去[Pytorch官网](https://pytorch.org/)下载Libtorch 压缩包，解压到本地目录即可，后面使用CMake来指向这里的路径就行。假如解压到`LIBTORCH_ROOT`目录，编译时添加下面的选项:
```bash
-DCMAKE_PREFIX_PATH=<LIBTORCH_ROOT>
```

## 3. 使用CMake 编译一个简单例子
这里写一个简单的Libtorch例子，创建一个5x5的矩阵，然后调用`einsum`函数来计算矩阵的迹（对角线元素的和）：
```cpp
// 引入Torch头文件，Tensor类在此头文件中，别的类会在另外的头文件中
#include <torch/torch.h>
#include <iostream>

int main() {
  // 使用arange构造一个一维向量，再用reshape变换到5x5的矩阵
  torch::Tensor foo = torch::arange(25).reshape({5, 5});

  // 计算矩阵的迹
  torch::Tensor bar  = torch::einsum("ii", foo);

  // 输出矩阵和对应的迹
  std::cout << "==> matrix is:\n " << foo << std::endl;
  std::cout << "==> trace of it is:\n " << bar << std::endl;
}
```
注意reshape中需要用花括号，因为C++没有tuple类型，Python中的`(5,5)`需要在C++中改写为`{5, 5}`。除此之外，是不是跟Python代码很相似？

记得保存上面的代码为`libtorch_trace.cpp`，因为CMake配置中需要写文件名。

然后在同级目录编写`CMakeLists.txt`文件:
```cmake
cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
project(libtorch_trace)

# 需要找到Libtorch
find_package(Torch REQUIRED)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")

add_executable(${PROJECT_NAME} libtorch_trace.cpp)
target_link_libraries(${PROJECT_NAME} "${TORCH_LIBRARIES}")

# Libtorch是基于C++14来实现的
set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 14)
```

然后执行下面的命令来编译:
```bash
mkdir build
cd build
# 如果是通过Pytorch
cmake -DCMAKE_PREFIX_PATH=`python -c 'import torch;print(torch.utils.cmake_prefix_path)'` ..
#下载的单独Libtorch
# cmake -DCMAKE_PREFIX_PATH=<LIBTORCH_ROOT> ..
make -j8
```

编译完成后使用下面的命令来执行可执行文件：
```bash
./libtorch_trace
```
结果如下：
```bash
==> matrix is:
   0   1   2   3   4
  5   6   7   8   9
 10  11  12  13  14
 15  16  17  18  19
 20  21  22  23  24
[ CPULongType{5,5} ]
==> trace of it is:
 60
[ CPULongType{} ]
```
那么我们的第一个例子就完成了。

## 4. 参考
+ <https://pytorch.org/cppdocs/installing.html>


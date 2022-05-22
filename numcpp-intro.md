---
title: NumPy的C++替代NumCpp使用教程
date: 2020-12-26 11:26:36
tags:
 - NumPy
 - NumCpp
 - C++
---

NumPy提供了很多开箱即用的函数，用处非常大，所以写C++的时候，让人无比怀念，要是有一个替代版本，就太好了。最近搜索发现， [NumCpp](https://github.com/dpilger26/NumCpp) 这是我想要的，而且因为是 `Header-only`的库，因此使用时不需要编译，直接添加到头文件包含目录即可，使用很方便。不过NumCpp使用了boost库，需要进行一些下载和配置，这里记录一下。
<!--more-->
总结下来下面是需要下载的东西，我写成了几行代码，在Ubuntu下测试是可以执行的：
```bash
mkdir includes
git clone https://github.com/dpilger26/NumCpp.git 
mv NumCpp/include includes/NumCpp
wget https://dl.bintray.com/boostorg/release/1.75.0/source/boost_1_75_0.zip
unzip boost_1_75_0.zip
mv boost_1_75_0/boost includes/NumCpp
```
这里我们创建了一个`includes`目录，用来存放NumCpp和Boost库的头文件，这里以现在 (2020-12-26) 最新的Boost 1.75.0 为例，后面boost库肯定会更新，可以从这里找到最新boost的下载地址：<https://www.boost.org/users/download>.


执行上面的命令后，就可以使用了NumCpp了，下面是一个使用示例：

```cpp
// 文件名：test_num_cpp.cpp

#include <iostream>

#include "NumCpp.hpp"

int main() {
        nc::NdArray<float> a = {{1, 2}, {3, 4}};
        nc::NdArray<float> b = {{1, 2}, {3, 4}};
        nc::NdArray<float> c = a * b;
        std::cout << c[0] << std::endl;

        return 0;
}
```
这个例子里面，简单地调用NumCpp最基本的类 `nc::NdArray`来进行两个2维数组的矩阵乘操作。
详细的教程参考：<https://github.com/dpilger26/NumCpp>.
接下来就是编译C++代码，这里以Linux下g++编译为例说明，需要注意的有2个点:
+ NumCpp只支持C++14以及以上版本，所以编译时需要加`--std=c++14`
+ 需要将NumCpp所在的目录添加到头文件包含指令`-I`里

具体如下:
```bash
g++ test_num_cpp.cpp --std=c++14 -Iincludes/
```
编译完后运行生成的可执行文件:
```bash
./a.out
```

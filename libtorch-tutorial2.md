---
title: libtorch系列教程2：torch::Tensor的使用
date: 2023-02-25 17:15:46
tags:
- Libtorch
- Pytorch
- C++
- Python
---
系列教程列表：
+ [Libtorch系列教程1：一个丝滑的C++ Tensor库](https://vra.github.io/2023/02/25/libtorch-tutorial1/) 
+ [Libtorch系列教程2：torch::Tensor的使用](https://vra.github.io/2023/02/25/libtorch-tutorial2/) 


这篇文章中，我们暂时忽略网络训练和推理，详细展开Libtorch中Tensor对象的使用，看看将Libtorch当作一个纯粹的Tensor库来使用时，有哪些注意事项。如有未涉及的内容，请访问Libtorch[官方文档](https://pytorch.org/cppdocs/)，通过搜索框获取更多的信息。Libtorch的环境搭建参考[上一篇文章](https://vra.github.io/2023/02/25/libtorch-tutorial1/)。
<!--more-->

## 1. torch::Tensor基本操作
Libtorch中的Tensor是与Pytorch中的Tensor对应的，使用方式上很类似，只在一些Python语法C++不支持的时候有些不同，例如slice操作。
使用Libtorch前需要包含 Libtorch 的头文件`torch/torch.h`:
```cpp
#include <torch/torch.h>
```
这篇文章用到的所有函数都在此头文件中声明，而且所有的函数namespace都是`torch`，因此都可以以`torch::xxx`的形式来调用。

### 1.1 Tensor创建
Tensor 创建的方式比较多，包括从字面量创建，从C++ 原生的数组创建，从vector创建，从Libtorch自带的函数创建等。

从字面量创建:
```cpp
torch::Tensor foo = torch::tensor({1.0, 2.0, 3.0, 4.0});
```

从C++ 原生的float数组创建，使用`from_blob`函数:
```cpp
float arr[] = {1.0, 2.0, 3.0, 4.0};
// 第二个参数表示创建的Tensor shape，会自动对原生数组进行reshape
torch::Tensor bar = torch::from_blob(arr, {1, 4}); // shape是[1, 4]
bar = torch::from_blob(arr, {2, 2}); // shape是[2, 2]
```
其中第二个参数表示创建的Tensor shape，会自动对原生数组进行reshape。

从vector 创建，使用`from_blob`函数:
```cpp
std::vector<float> v = {1.0, 2.0, 3.0, 4.0};
bar = torch::from_blob(v.data(), {2, 2});
```
还可以用Libtorch的函数创建，跟Numpy和Pytorch类似:
```cpp
foo = torch::arange(4);
foo = torch::eye(2);
foo = torch::ones(2);
bar = torch::ones_like(foo);
foo = torch::rand(4);
foo = torch::randn(4);
foo = torch::zeros(2);
bar = torch::zeros_like(foo);
```

创建好以后，Tensor对应可以直接用`std::cout`来输出:
```cpp
torch::Tensor foo = torch::tensor({1.0, 2.0, 3.0, 4.0});
std::cout <<"==> foo is:\n" << foo << std::endl;
```
输出如下：
```bash
==> foo is:
 1
 2
 3
 4
[ CPUFloatType{4} ]
```
可以看到最后打印了Tensor的类型。

### 1.2 Tensor对象的属性函数
创建Tensor后，我们还需要看到它的一些属性，判断是否跟预期相符。注意Libtorch的Tensor是没有公开可访问的属性attribute的，Tensor信息需要属性函数来获取。常见的属性函数包括:
+ dim(): Tensor的维度
+ sizes(): 跟Pytorch中的shape属性一样
+ size(n): 第N个维度的shape
+ numel(): 总的元素数目，sizes中的每个元素相乘
+ dtype(): 数据类型
+ device(): Tensor所在的设备类型，CPU, CUDA, MPS等。

使用方式如下:
```cpp
// Tensor 属性函数
torch::Tensor foo = torch::randn({1, 3, 224, 224});
auto dim = foo.dim(); // 4
auto sizes = foo.sizes(); // [1, 3, 224, 224]
auto size_0 = foo.size(0); // 1
auto numel = foo.numel(); // 150528
auto dtype = foo.dtype(); // float
auto scalar_type = foo.scalar_type(); // Float
auto device = foo.device(); // cpu
```

### 1.3 Tensor对象的索引
Tensor 默认是支持`[]`操作符的，因此可以使用这样的方式来获取元素：
```cpp
auto foo = torch::randn({1, 2, 3, 4});
float value = foo[0][1][2][2];
```
另一种方式是用Tensor对象的`index`函数，它的优势是支持slice。
对于单个元素，可以类似Pytorch中，直接用`index({i, j, k})`的方式来索引：
```cpp
auto foo = torch::randn({1, 2, 3, 4});
float value = foo.index({0, 1, 2, 2});
```
那么python中很常用的slice呢？例如`foo[..., :2, 1:, :-1]`，该怎么在Libtorch中表示？
这里需要用到`torch::indexing::Slice` 对象，来实现Python中的Slice，看看下面的例子你就明白了：
```cpp
using namespace torch::indexing;

auto foo = torch::randn({1, 2, 3, 4});
// 等效于Python中的foo[:, 0:1, 2:, :-1]
auto bar = foo.index({Slice(), Slice(0, 1), Slice(2, None), Slice(None, -1)});
```
应该是能满足Python中slice同样的使用场景。

### 1.4 更新Tensor中元素的值

有了索引之后，我们就可以更新Tensor的值了：
```cpp
torch::Tensor foo = torch::tensor({1.0, 2.0, 3.0, 4.0});
foo[0] = 10.0;
foo.index({0}) = 2.0;
```
但还没找到用给部分Tensor元素赋值的方法，类似Python中的`foo[:2] = bar`，欢迎补充。

### 1.5 获取Tensor中的数据
Tensor是一个Libtorch的对象，那怎么把它中的数据拿出来保存到文件中或传给别的函数呢？
使用`data_ptr`函数就可以:
```cpp
torch::Tensor foo = torch::randn({3, 3});
float* data = foo.data_ptr<float>();
```
对于单个元素的Tensor，还可以用`item`函数得到具体的数值:
```cpp
torch::Tensor one_element_tensor = foo.index({Slice(), Slice(0, 1), Slice(0, 1), Slice(0, 1)});
float value = one_element_tensor.item<float>();
```

### 1.6 数据类型
Libtorch中支持float16, float32, float64, int8, int16, int32, uint8这几类的Tensor数据类型，可以用`to`函数来进行类型转换：
```cpp
// 数据类型, 参见 https://pytorch.org/cppdocs/api/file_torch_csrc_api_include_torch_types.h.html#variables
bar = foo.to(torch::kF16);
bar = foo.to(torch::kF32);
bar = foo.to(torch::kF64);
bar = foo.to(torch::kFloat16);
bar = foo.to(torch::kFloat32);
bar = foo.to(torch::kFloat64);
bar = foo.to(torch::kI8);
bar = foo.to(torch::kI16);
bar = foo.to(torch::kI32);
bar = foo.to(torch::kI64);
bar = foo.to(torch::kInt8);
bar = foo.to(torch::kInt16);
bar = foo.to(torch::kInt32);
bar = foo.to(torch::kInt64);
bar = foo.to(torch::kU8);
bar = foo.to(torch::kUInt8);
```
全部数据类型，参见官方文档的[数据类型页面](https://pytorch.org/cppdocs/api/file_torch_csrc_api_include_torch_types.h.html#variables)。

### 1.7 设备类型
设备类型是Tensor保存的设备的种类。由于Libtorch不仅仅支持CPU，还支持各种类型的GPU，因此有很多设备类型。

所有的设备类型参见[这里](https://pytorch.org/cppdocs/api/file_c10_core_DeviceType.h.html#variables)。
需要注意的是，设备是跟编译时的配置，机器是否支持强相关的，而且某些设备支持并不好，例如我想用下面的代码将CPU上的Tensor转移到MPS上：
```cpp
auto foo = torch::randn({3, 3});
auto bar = foo.to(torch::kMPS);
```
编译是没有问题的，但运行时会报下面的错:
> libc++abi: terminating with uncaught exception of type c10::TypeError: Cannot convert a MPS Tensor to float64 dtype as the MPS framework doesn't support float64. Please use float32 instead.

提示说MPS不支持float64，但我打印`foo`的类型，它其实是float32，本身报错比较奇怪，搜了一圈也没找到怎么解决。

### 1.8 Tensor 变形函数
很多时候我们需要将Tensor进行形状的修改，这方面Libtorch支持的比较好，这些操作都支持:
+ reshape
+ flatten
+ squeeze
+ unsqueeze
+ transpose
+ cat/concat/concatenate

而且支持`torch::reshape`这种静态函数和`tensor.reshape`这种对象函数。下面是一些例子:
```cpp
// 变形操作
bar = foo.reshape({2, -1});
bar = foo.flatten();
bar = foo.squeeze();
bar = foo.unsqueeze(0);
bar = torch::unsqueeze(foo, -1);
bar = foo.transpose(0, 1).transpose(2, 3).transpose(3, 1);
bar = torch::transpose(foo, 0, 1);
bar = torch::cat({foo, foo}, 2);
```
一个比较特殊的地方是transpose只支持两个轴的交换，多个轴的交换需要调用多次来实现。

### 1.9 Tensor之间的操作函数
Tensor库中，Tensor和Tensor之间的操作是很常见的，比如求矩阵相乘，内积外积等，有内置的函数支持能避免很多额外的开发工作。这里是一些例子:
```cpp
foo = torch::randn({3, 3});
bar = torch::matmul(foo, foo);
bar = foo.matmul(foo);
bar = torch::cross(foo, foo);
bar = torch::mul(foo, foo);
```
### 1.10 线性代数相关函数
`torch::linalg` namespace中包含常见的线性代数操作，几个简单的使用例子:
```cpp
bar = torch::linalg::inv(foo);
bar = torch::linalg::norm(foo, 2, {0, 1}, false, torch::nullopt);
```
所有支持的函数详见[官方文档](https://pytorch.org/cppdocs/api/file_torch_csrc_api_include_torch_linalg.h.html#file-torch-csrc-api-include-torch-linalg-h)

### 1.11 神经网络相关函数
神经网络是torch的核心模块，常见的一些激活函数，卷积层都可以以函数的形式作用在Tensor上，这里写几个简单的例子：
```cpp
bar = torch::softmax(foo, -1);
bar = torch::sigmoid(foo);
bar = torch::relu(foo);
bar = torch::gelu(foo);
```

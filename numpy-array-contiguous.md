---
title: 从Numpy中的ascontiguousarray说起
date: 2019-03-18 23:02:58
tags:
 - Python
 - Numpy
 - 总结 
---
## 1. 概述
在使用Numpy的时候，有时候会遇到下面的错误：
```bash
AttributeError: incompatible shape for a non-contiguous array
```
看报错的字面意思，好像是不连续数组的shape不兼容。

有的时候，在看别人代码时会时不时看到`ascontiguous()`这样的一个函数，查[文档](https://docs.scipy.org/doc/numpy/reference/generated/numpy.ascontiguousarray.html)会发现函数说明只有一句话：“Return a contiguous array (ndim >= 1) in memory (C order).”

光靠这些信息，似乎没能道出Numpy里面contiguous array和non-contiguous array有什么区别，以及为什么需要进行`ascontiguous`操作？带着这些疑问，我搜了比较多的资料，在stack overflow上发现一个比较详细的[回答](https://stackoverflow.com/questions/26998223/what-is-the-difference-between-contiguous-and-non-contiguous-arrays)，简单明白地将Numpy里面的数组的连续性问题解释清楚了，因此这里翻译过来，希望能帮助到别的有同样疑问的小伙伴。

## 2. 额外知识： C order vs Fortran order
所谓`C order`，指的是行优先的顺序（Row-major Order)，即内存中同行的存在一起，而`Fortran Order`则指的是列优先的顺序（Column-major Order)，即内存中同列的存在一起。这种命名方式是根据C语言和Fortran语言中数组在内存中的存储方式不同而来的。Pascal, C，C++，Python都是行优先存储的，而Fortran，MatLab是列优先存储的。
<!--more-->

## 3. 译文
所谓`contiguous array`，指的是数组在内存中存放的地址也是连续的（注意内存地址实际是一维的），即访问数组中的下一个元素，直接移动到内存中的下一个地址就可以。

考虑一个2维数组`arr = np.arange(12).reshape(3,4)`。这个数组看起来结构是这样的：
![](/imgs/numpy_ascontiguous_2.png)

在计算机的内存里，数组`arr`实际存储是像下图所示的：
![](/imgs/numpy_ascontiguous_3.png)

这意味着`arr`是`C连续的`（`C contiguous`)的，因为在内存是行优先的，即某个元素在内存中的下一个位置存储的是它同行的下一个值。

如果想要向下移动一列，则只需要跳过3个块既可（例如，从0到4只需要跳过1,2和3）。

上述数组的转置`arr.T`则没有了C连续特性，因为同一行中的相邻元素现在并不是在内存中相邻存储的了:
![](/imgs/numpy_ascontiguous_1.png)
这时候`arr.T`变成了`Fortran 连续的`（`Fortran contiguous`），因为相邻列中的元素在内存中相邻存储的了。

从性能上来说，获取内存中相邻的地址比不相邻的地址速度要快很多（从RAM读取一个数值的时候可以连着一起读一块地址中的数值，并且可以保存在Cache中）。这意味着对连续数组的操作会快很多。

由于`arr`是C连续的，因此对其进行行操作比进行列操作速度要快，例如，通常来说
```python
np.sum(arr, axis=1) # 按行求和
```
会比
```python
np.sum(arr, axis=0) # 按列求和
```
稍微快些。

同理，在`arr.T`上，列操作比行操作会快些。

## 4. 补充
Numpy中，随机初始化的数组默认都是C连续的，经过不规则的`slice`操作，则会改变连续性，可能会变成既不是C连续，也不是Fortran连续的。

Numpy可以通过`.flags`熟悉查看一个数组是C连续还是Fortran连续的
```python
>>> import numpy as np
>>> arr = np.arange(12).reshape(3, 4)
>>> arr.flags
  C_CONTIGUOUS : True
  F_CONTIGUOUS : False
  OWNDATA : False
  WRITEABLE : True
  ALIGNED : True
  WRITEBACKIFCOPY : False
  UPDATEIFCOPY : False
```
从输出可以看到数组`arr`是C连续的。

对`arr`进行按列的`slice`操作，不改变每行的值，则还是C连续的：
```python
>>> arr
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])
>>> arr1 = arr[:3, :]
>>> arr1
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])
>>> arr1.flags
  C_CONTIGUOUS : True
  F_CONTIGUOUS : False
  OWNDATA : False
  WRITEABLE : True
  ALIGNED : True
  WRITEBACKIFCOPY : False
  UPDATEIFCOPY : False
```
如果进行在行上的`slice`，则会改变连续性，成为既不C连续，也不Fortran连续的：
```python
>>> arr1 = arr[:, 1:3]
>>> arr1.flags
  C_CONTIGUOUS : False
  F_CONTIGUOUS : False
  OWNDATA : False
  WRITEABLE : True
  ALIGNED : True
  WRITEBACKIFCOPY : False
  UPDATEIFCOPY : False
```
此时利用`ascontiguousarray`函数，可以将其变为连续的：
```python
>>> arr2 = np.ascontiguousarray(arr1)
>>> arr2.flags
  C_CONTIGUOUS : True
  F_CONTIGUOUS : False
  OWNDATA : True
  WRITEABLE : True
  ALIGNED : True
  WRITEBACKIFCOPY : False
  UPDATEIFCOPY : False
```
可以这样认为，`ascontiguousarray`函数将一个内存不连续存储的数组转换为内存连续存储的数组，使得运行速度更快。

---
title: 从Python传递参数到C++
date: 2022-03-20 21:05:30
tags:
- C++
- Python
- Linux
---
## 概述
有些场景下，需要将Python里面计算得到的参数或者结果传入到C++来进行工程部署。一个常见问题是，Python该以什么格式 (二进制还是文本) 保存这些参数，然后从C++代码里面来读取呢，各有什么优劣？这里我们简单实验一下，并写一些趁手的代码，供查阅。
<!--more-->

## 二进制格式和文本格式对比
假设我们有一组参数是存储在Numpy的`ndarray`格式中的，为了在C++中使用，我们需要保存它们到硬盘的文件中。一般有两种保存方法：二进制文件保存和文本文件保存。

假设我们有一个1024x1024的浮点型参数待保存：
```python
params = np.random.rand(1024, 1024).astype('float32')
```
二进制保存很简单，直接调用Numpy的`tofile`文件即可：
```python
params.tofile("params.bin")
```
如果用文本文件保存，有两种保存方式，分别为调用`savetxt`函数和将每个值转换为`str`并用分隔符分开依次存入文件:
```python
# 文本文件保存方式1
np.savetxt("params_1.txt", params)

# 文本文件保存方式2
delimiter = " "
with open("params_2.txt", "w") as f:
    for p in params:
        f.write(str(p) + delimiter)
```
猜猜看这三种情况分别大小是多少？

结论如下：
```bash
4.0M params.bin
25M params_1.txt
11M params_2.txt
```
可以看到，二进制格式存储空间是最小的，分别是两种文本形式存储空间的16%和36%，存储压缩比例还是比较明显的。

因此推荐以二进制形式存储, 存储脚本简单总结如下:
```python
import numpy as np

# rand默认格式是float64，我们使用float32就可以
params = np.random.rand(1024, 1024).astype("float32")

# 拉平成一维，为了在C++里面方便处理
params = params.flatten()

params.tofile("params.bin")
```

## C++ 读取二进制文件

C++ 去读二进制的代码如下：
```cpp
#include <fstream>
#include <iostream>
#include <string>

void read_binary(const std::string &file_path, float *data, int size) {
  std::ifstream in_file;
  in_file.open(file_path, std::ios::binary | std::ios::in);
  in_file.read((char *)data, size * sizeof(float));
  in_file.close();
}

int main() {
  std::string file_path = "params.bin";
  int size = 1024 * 1024;

  // 使用stack上空间来创建数组，有大小限制，不推荐
  // float params[size];

  // 使用new来构建heap上空间, 无大小限制，但需要自己释放内存
  float *params = new float[size];
  read_binary(file_path, params, size);

  // 打印前10个参数
  for (int i = 0; i < 10; i++) {
    std::cout << params[i] << std::endl;
  }

  delete[] params;
}
```
注意新建数组的时候，有在栈上或者堆上构建两种方式，栈上构建有大小限制，如果数组维度太大就会报错，如下面的代码:
```cpp
#include <iostream>
int main() {
        int arr[1024*1024*1024];

        return 0;
}
```
运行会报错:
```bash
$ g++ stack_over.cpp && ./a.out
[1]    89415 segmentation fault  ./a.out
```
因此推荐用堆上创建数组，详见上述代码的注释。

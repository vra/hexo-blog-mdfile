---
title: C++中使用pytorch保存的tensor
date: 2021-03-21 18:19:28
tags:
 - Pytorch
 - Libtorch
 - Python
 - C++
---
## 概述
最近在学习Libtorch——即Pytorch的C++版本，需要使用 Pytorch 导出的 tensor 以便对模型进行 debug。下面是转换代码，总体原理是将 tensor 转换为二进制数据，再在 C++ 里面读入。

<!--more-->

下面是 Pytorch 中的导出 tensor 示例：
```python
import io

import torch


def save_tensor(device):
    my_tensor = torch.rand(3, 3).to(device);
    print("[python] my_tensor: ", my_tensor)
    f = io.BytesIO()
    torch.save(my_tensor, f, _use_new_zipfile_serialization=True)
    with open('my_tensor_%s.pt' % device, "wb") as out_f:
        # Copy the BytesIO stream to the output file
        out_f.write(f.getbuffer())


if __name__ == '__main__':
    save_tensor('cpu')
```
这里以导出 cpu tensor 为例，cuda tensor 也是同理。

在 C++ 中的调用示例如下：
```cpp
#include <iostream>
#include <torch/torch.h>

std::vector<char> get_the_bytes(std::string filename) {
    std::ifstream input(filename, std::ios::binary);
    std::vector<char> bytes(
        (std::istreambuf_iterator<char>(input)),
        (std::istreambuf_iterator<char>()));

    input.close();
    return bytes;
}

int main()
{
    std::vector<char> f = get_the_bytes("my_tensor_cpu.pt");
    torch::IValue x = torch::pickle_load(f);
    torch::Tensor my_tensor = x.toTensor();
    std::cout << "[cpp] my_tensor: " << my_tensor << std::endl;

    return 0;
}
```

注意事项：
1. torch的Python和C++版本需要保持一致，否则转换可能不成功.

## 题外话
最近在学习Libtorch——即Pytorch的C++版本，发现使用起来异常的丝滑，写C++有了Python的体验，妙不可言。
后面会更新一些关于libtorch使用的文章，敬请关注。

## 参考
1. <https://discuss.pytorch.org/t/how-to-load-python-tensor-in-c/88813>

---
title: Pytorch模型转ONNX时cross操作不支持的解决方法
date: 2022-03-20 18:34:46
tags:
- Pytorch
- Python
- ONNX
- Deep Learning
- Torchscript
---
## 概述
Pytorch很灵活，支持各种OP和Python的动态语法。但是转换到onnx的时候，有些OP（目前）并不支持，比如`torch.cross`。这里以一个最小化的例子来演示这个过程，以及对应的解决办法。
<!--more-->

## 一个例子
考虑下面这个简单的Pytorch转ONNX的例子：
```python
# file name: pytorch_cross_to_onnx.py
import torch
import torch.nn as nn


class MyModel(nn.Module):
    def __init__(self):
        super(MyModel, self).__init__()
        self.conv = nn.Conv2d(3, 10, 3, stride=1)

    def forward(self, x):
        x = torch.cross(x, x)
        y = self.conv(x)

        return y


model = MyModel()

dummy_input = torch.randn(1, 3, 224, 224, device="cpu")
input_names = ["x"]
output_names = ["y"]

# opset_version 选择范围：[7,15]
torch.onnx.export(
    model,
    dummy_input,
    "my_model.onnx",
    input_names=input_names,
    output_names=output_names,
    opset_version=14
)
```
运行这个脚本，会报下面的错误：
```bash
$ python3 pytorch_cross_to_onnx.py
Traceback (most recent call last):
  File "pytorch_cross.py", line 25, in <module>
    torch.onnx.export(model, dummy_input, "my_model.onnx", input_names=input_names, output_names=output_names, opset_version=14)
  File "/usr/local/lib/python3.7/site-packages/torch/onnx/__init__.py", line 320, in export
    custom_opsets, enable_onnx_checker, use_external_data_format)
  File "/usr/local/lib/python3.7/site-packages/torch/onnx/utils.py", line 111, in export
    custom_opsets=custom_opsets, use_external_data_format=use_external_data_format)
  File "/usr/local/lib/python3.7/site-packages/torch/onnx/utils.py", line 729, in _export
    dynamic_axes=dynamic_axes)
  File "/usr/local/lib/python3.7/site-packages/torch/onnx/utils.py", line 501, in _model_to_graph
    module=module)
  File "/usr/local/lib/python3.7/site-packages/torch/onnx/utils.py", line 216, in _optimize_graph
    graph = torch._C._jit_pass_onnx(graph, operator_export_type)
  File "/usr/local/lib/python3.7/site-packages/torch/onnx/__init__.py", line 373, in _run_symbolic_function
    return utils._run_symbolic_function(*args, **kwargs)
  File "/usr/local/lib/python3.7/site-packages/torch/onnx/utils.py", line 1028, in _run_symbolic_function
    symbolic_fn = _find_symbolic_in_registry(domain, op_name, opset_version, operator_export_type)
  File "/usr/local/lib/python3.7/site-packages/torch/onnx/utils.py", line 982, in _find_symbolic_in_registry
    return sym_registry.get_registered_op(op_name, domain, opset_version)
  File "/usr/local/lib/python3.7/site-packages/torch/onnx/symbolic_registry.py", line 125, in get_registered_op
    raise RuntimeError(msg)
RuntimeError: Exporting the operator cross to ONNX opset version 14 is not supported. Please feel free to request support or submit a pull request on PyTorch GitHub.
```
注意最后一句的报错:
```bash
RuntimeError: Exporting the operator cross to ONNX opset version 14 is not supported. Please feel free to request support or submit a pull request on PyTorch GitHub.
```
也就是说目前版本是不支持`torch.cross`转onnx的，同时提示你"feel free" 去Pytorch 的 GitHub 上提交/贡献一个转换操作。不过2020年03月就有人提了[issue](https://github.com/onnx/onnx/issues/2683)，至今仍没有g官方的解决方案。

## 解决办法
上面的issue里有人给出了解决思路，就是用元素相乘替代`cross`操作。具体来说，实现如下：
```python
def my_cross(x, y, dim=1):
    assert x.dim() == y.dim() and dim < x.dim()

    return torch.stack(
        (
            x[:, 1, ...] * y[:, 2, ...] - x[:, 2, ...] * y[:, 1, ...],
            x[:, 2, ...] * y[:, 0, ...] - x[:, 0, ...] * y[:, 2, ...],
            x[:, 0, ...] * y[:, 1, ...] - x[:, 1, ...] * y[:, 0, ...],
        ),
        dim=dim,
    )
```
**注意：这里是以dim=1为例写的实现，如果是在别的维度进行cross操作，需要修改dim参数，同时修改对应stack的维度。**

同时在Pytorch doc网站上看到，如果`torch.cross`不指定`dim`参数的话，默认是从前往后找第一个维度为3的维度，因此这个可能是你所不期望的，建议显式指定这个参数。

因此总结下来，下面是修改后的代码:
```python
import torch
import torch.nn as nn


def my_cross(x, y, dim=1):
    assert x.dim() == y.dim() and dim < x.dim()

    return torch.stack(
        (
            x[:, 1, ...] * y[:, 2, ...] - x[:, 2, ...] * y[:, 1, ...],
            x[:, 2, ...] * y[:, 0, ...] - x[:, 0, ...] * y[:, 2, ...],
            x[:, 0, ...] * y[:, 1, ...] - x[:, 1, ...] * y[:, 0, ...],
        ),
        dim=dim,
    )


class MyModel(nn.Module):
    def __init__(self):
        super(MyModel, self).__init__()
        self.conv = nn.Conv2d(3, 10, 3, stride=1)

    def forward(self, x):
        # x = torch.cross(x, x)
        x = my_cross(x, x)
        y = self.conv(x)

        return y


model = MyModel()

dummy_input = torch.randn(1, 3, 224, 224, device="cpu")
output = model(dummy_input)
input_names = ["x"]
output_names = ["y"]

# opset_version 选择范围：[7,15]
torch.onnx.export(
    model,
    dummy_input,
    "my_model.onnx",
    input_names=input_names,
    output_names=output_names,
    opset_version=14,
)
```
为了验证我们的实现与Pytorch的实现是否一致，可以用下面的函数验证:
```python
def test_torch_cross_and_my_cross():
    x = torch.randn(10, 3, 10, 10)
    y = torch.randn(10, 3, 10, 10)
    print("my_cross == torch.cross:", torch.allclose(torch.cross(x, y), my_cross(x, y)))
```
执行后输出如下:
```bash
my_cross == torch.cross: True
```
说明这个实现是正确的。


## 参考
1. <https://github.com/onnx/onnx/issues/2683>
2. <https://pytorch.org/docs/stable/generated/torch.cross.html>

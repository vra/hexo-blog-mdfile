---
title: Pytorch 拟合多项式的例子
date: 2022-06-26 11:20:15
tags:
- Deep Learning
- Pytorch
---

## 1. 概述
Pytorch包含了Linear层，可以用来拟合`y = w * x + b` 形式的函数，其中`w`和`bias`就是Linear层的weights和bias。这里写个拟合一次多项式的简单demo，作为一个小实验。
<!--more-->

## 2. 拟合一次多项式
采用下面的代码，我们设计了一个包含一个线性层的网络，通过给它feed随机构造的数据(y = 1.233 * x + 0.988)，结合梯度下降算法和MSE loss惩罚函数，让它学习数据的构造参数:
```python
import torch
import torch.nn as nn
import torch.optim as optim
import torch.nn.functional as F


class Model(nn.Module):
    def __init__(self):
        super(Model, self).__init__()
        self.linear = nn.Linear(1, 1)

    def forward(self, x):
        x = self.linear(x)
        return x


def run():
    torch.manual_seed(1024)

    model = Model()
    model.train()
    optimizer = optim.SGD(model.parameters(), lr=1e-2)

    w = 1.233
    b = 0.988
	num_iteration = 5000
    for i in range(num_iteration):
        optimizer.zero_grad()
        x = torch.rand(1)
        y = w * x + b
        pred = model(x)

        loss = F.mse_loss(y, pred)
        loss.backward()
        optimizer.step()

    for name, param in model.named_parameters():
        print(f"{name}={param.data.numpy().squeeze():.3f}")


if __name__ == "__main__":
    run()
```
运行这个脚本的输出结果如下：
```bash
linear.weight=1.233
linear.bias=0.988
```
可以看到，经过5000次的迭代，网络能成功地学习到数据构造过程中的w和b参数, 这个小网络现在可以用来替代线性回归机器学习算法了!

如果迭代周期太小则可能收敛不到我们预设的参数，可以手动修改迭代次数`num_iteration`为2000查看结果。

## 3. 如果重复Linear层会发生什么？

如果我们把同一个linear层重复执行两次，会有什么结果呢？也就是网络定义修改如下：
```python
class Model(nn.Module):
    def __init__(self):
        super(Model, self).__init__()
        self.linear = nn.Linear(1, 1)

    def forward(self, x):
        x = self.linear(x)
        x = self.linear(x)
        return x
```
这里调用了两次同一个linear层，因此相当于 `y = w * ( w * x + b) + b`，也就是一次forward更新两次参数，也可以理解成两个共享参数的线性层。
完整的示例代码如下：
```python
import torch
import torch.nn as nn
import torch.optim as optim
import torch.nn.functional as F


class Model(nn.Module):
    def __init__(self):
        super(Model, self).__init__()
        self.linear = nn.Linear(1, 1)

    def forward(self, x):
        x = self.linear(x)
        x = self.linear(x)
        return x


def run():
    torch.manual_seed(1024)

    model = Model()
    model.train()
    optimizer = optim.SGD(model.parameters(), lr=1e-2)

    w = 1.233
    b = 0.988
    num_iteration = 5000
    for i in range(num_iteration):
        optimizer.zero_grad()
        x = torch.rand(1)
        y = w * x + b
        pred = model(x)

        loss = F.mse_loss(y, pred)
        loss.backward()
        optimizer.step()

    for name, param in model.named_parameters():
        print(f"{name}={param.data.numpy().squeeze():.3f}")


if __name__ == "__main__":
    run()
```

同样的，通过我们构造 y = 1.233 * x + 0.998的数据，带入 y = w * ( w * x + b) + b，可以得到一组解 `w=1.110, b=0.468`,这与我们网络运行得到的结果是一致的：
```bash
linear.weight=1.110
linear.bias=0.468
```
同时也有一个问题：为什么没得到w为负数的另一组解呢？这是因为我这里为了保证复现性，手动设置了随机数种子为1024，设置为别的值应该可以得到另一组参数，欢迎尝试。

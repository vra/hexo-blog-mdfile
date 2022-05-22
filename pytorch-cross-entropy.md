---
title: Pytorch使用交叉熵损失时的一个坑
date: 2021-08-21 07:03:13
tags:
- Deep Learning
- Pytorch
- Python
---
在Pytorch里面使用交叉熵loss函数的时候，发现结果最是比较差，通过搜索才发现这样一段话：
> You should pass raw logits to nn.CrossEntropyLoss, since the function itself applies F.log_softmax and nn.NLLLoss() on the input.

也就是用交叉熵损失的时候，不能在网络的最后用 `log_softmax` 或者 `Softmax`层，因为交叉熵损失相当与是 `log_softmax` + `NLLLos`的组合。

如果网络最后用了Softmax层的话，需要使用 `NLLLoss` 或者 `MSE loss`。


## 参考:
1. <https://discuss.pytorch.org/t/logsoftmax-vs-softmax/21386/9>

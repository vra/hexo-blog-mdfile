---
title: Python中将图像切分为小的patch
date: 2022-10-15 07:51:36
tags:
- Python
- Numpy
- Pytorch
---

### 问题定义
假如有张1000x1000的图像，我们要将它切成20x20的小patch，该怎么处理呢？
最简单的方法就是采用两重for循环，每次计算小patch对应的下标，在原图上进行crop:
```python
import numpy as np

size = 1000
ncols = 20
nrows = 20
img = np.random.rand(size, size)

patches = []

for i in range(size//ncols):
	for j in range(size//nrows):
		patch = img[ncols*i:ncols*(i+1), nrows*j:nrows*(j+1)]
		patches.append(patch)

patches = np.array(patches)
```

但这样总共需要循环50*x50=2500次，而我们知道 Python 的 for 循环比较慢，因此整体开销还是比较大的，有没有更快的方式呢？

<!--more-->

### reshape + swapaxes

搜索发现可以使用 reshape + swapaxes函数的组合来完成这个功能:
```python
import numpy as np

size = 1000
ncols = 20
nrows = 20
img = np.random.rand(size, size)

patches = img.reshape(size // ncols, ncols, -1, nrows).swapaxes(1, 2).reshape(-1, ncols, nrows)
```

完整对比代码如下：
```python
import time

import numpy as np


size = 1000
ncols = 20
nrows = 20

# 统计100次耗时
for i in range(100):
    img = np.random.rand(size, size)
    t0 = time.time()
    patches0 = img.reshape(size // ncols, ncols, -1, nrows).swapaxes(1, 2).reshape(-1, ncols, nrows)

    t1 = time.time()
    d1 = t1 - t0

    patches = []
    for i in range(size//ncols):
        for j in range(size//nrows):
            patch = img[ncols*i:ncols*(i+1), nrows*j:nrows*(j+1)]
            patches.append(patch)

    patches1 = np.array(patches)
    t2 = time.time()
    d2 = t2 - t1

    print('time ratio:', d2/d1)
    print('diff:', (patches0-patches1).sum())
```

实际测试对于1000x1000的图像，采用reshape + swapaxes 要比循环快大约4倍。
```bash
time ratio: 4.684571428571428
diff: 0.0
time ratio: 4.806614785992218
diff: 0.0
time ratio: 4.696482035928144
diff: 0.0
time ratio: 3.00382226469183
diff: 0.0
time ratio: 3.710854363028276
diff: 0.0
...
```


### Pytorch中的实现？
Pytorch相比numpy，又增加了许多操作tensor的函数，因此实现方式会更多，这里大概列一下几种实现，具体函数可以查询 Pytorch 的文档:
```python
patches1 = img.unfold(0, ncols, nrows).unfold(1, ncols, nrows).reshape(-1, ncols, nrows)
patches2 = img.reshape(size//ncols, ncols, -1, nrows).swapaxes(1, 2).reshape(-1, ncols, nrows)
patches3 = img.reshape(size//ncols, ncols, -1, nrows).permute(0, 2, 1, 3).reshape(-1, ncols, nrows)
```

### 其他相关操作
ShuffleNet中的ShuffleBlock中的channel shuffle也是通过reshape+维度变换来完成的，可以参考[这里](https://github.com/MegEngine/Models/blob/master/official/vision/classification/shufflenet/model.py#L98) 和[这里](https://iq.opengenus.org/shufflenet-implementation-using-pytorch/)的实现。

另外之前一篇做分割的论文[DUC](https://arxiv.org/abs/1702.08502)里面也用到了类似的把图像特征重排列来Upsample的操作，[搜索了下](https://github.com/ycszen/pytorch-segmentation/blob/master/duc.py#L18)对应的实现，是用Pytorch的PixelShuffle来做的，具体用法参考[文档](https://pytorch.org/docs/stable/generated/torch.nn.PixelShuffle.html)，还有个匹配的PixelUnShuffle来进行逆向操作。


### 参考
1. <https://stackoverflow.com/questions/16856788/slice-2d-array-into-smaller-2d-arrays/16858283#16858283>


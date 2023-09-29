---
title: 一个简单好用的Python并行函数
date: 2023-08-12 23:43:21
tags:
 - Python
 - mmengine
---
### 1. 背景
用Python跑有大量数据的任务的时候，启用多进程加速效果明显。但因为我之前在使用Python的多进程库时总遇到卡住的问题，后来对这块避而远之，总是用别的方法来加速。最近发现OpenMMLab的一些库提供了多进程并行的函数功能，简单好用。比如一个简单的toy例子，OpenCV读图像，resize然后保存，在8个CPU核的 Mac 上，加速比能达到3.4倍(45ms vs 13ms)，也就是以前要跑3个多小时的任务，现在1个小时就能搞定，省了不少时间，更多实际例子也证明了这个函数的加速效果，还是挺实用的。这里写个教程，希望也能方便到别的有同样需要的人，当然同类型的库应该也有很多，这里只是取一瓢饮。
<!--more-->

### 2. 函数实现
具体实现是[mmengine](https://github.com/open-mmlab/mmengine)中的[track_parallel_progress](https://github.com/open-mmlab/mmengine/blob/main/mmengine/utils/progressbar.py#L109)函数，它底层也是调用了Python系统库的[multiprocessing](https://docs.python.org/3/library/multiprocessing.html)，进行多进程加速脚本的运行。所以原理上来说我们也可以不用这个函数，自己写multiprocessing调用代码。但mmengine的这个封装，给我们省去了写multiprocessing比较复杂的调度代码的时间，拿来直接用还是能加速代码的开发节奏。

大致的调用框架:
```python

from functools import wraps
import mmengine


def mmengine_track_func(func):
    # wraps的作用是将装饰器的信息都传递给被装饰的函数，
    # 参考：https://stackoverflow.com/a/309000
    @wraps(func)
    def wrapped_func(args):
        return func(*args)

    return wrapped_func


@mmengine_track_func
def your_func(arg1, arg2):
    # your code here
    return results

# 进程数
NUM_PROC = 8

# 构造调用参数
params = [(arg1, arg2) for arg1, arg2 in zip(arg1_list, arg2_list)]

# 调用mmengine封装好的多进程函数
results = mmengine.track_parallel_progress(your_func, params, nproc=NUM_PROC)
```
使用时需要先 `pip install mmengine`来安装依赖库 mmengine。

然后这里构造了一个装饰器`mmengine_track_func`，对实际调用的函数`your_func`进行封装。其中用到了functools中的wraps函数，它的作用是将装饰器的信息都传递给被装饰的函数，具体例子可以参考这个[回答](https://stackoverflow.com/a/309000)。

实际使用时`mmengine_track_func` 不需要修改，直接采用这种形式。

然后是设置进程数，构造你自己函数的参数，再调用`mmengine.track_parallel_progress` 即可，它的必需的三个参数分别是:
1. 你的函数名
2. 函数参数list
3. 设置的进程数

别的非必需参数可以参考[源码](https://github.com/open-mmlab/mmengine/blob/main/mmengine/utils/progressbar.py#L109)。

### 3. toy 例子

这里举一个简单的伪造例子，读取本地某个目录下的png图像，将它们都缩放到200x200，再保存到本地。完整代码如下:
```python
from functools import wraps
import glob
import os
import time

import cv2
import mmengine
from tqdm import tqdm


def mmengine_track_func(func):
    # wraps的作用是将装饰器的信息都传递给被装饰的函数，
    # 参考：https://stackoverflow.com/a/309000
    @wraps(func)
    def wrapped_func(args):
        return func(*args)

    return wrapped_func


@mmengine_track_func
def run(idx, img_path):
    img = cv2.imread(img_path)
    img = cv2.resize(img, (200, 200))

    op = f"{idx}.jpg"
    cv2.imwrite(op, img)


if __name__ == "__main__":
    # 获取所有图片路径
    img_paths = glob.glob("/path/to/folder/*.png")

    # 测试开多线程版本，耗时 13ms
    params = [(idx, img_path) for idx, img_path in enumerate(img_paths)]
    mmengine.track_parallel_progress(run, params, nproc=8)

    # 测试不开多线程版本，耗时45ms
    t0 = time.time()
    for idx, ip in tqdm(enumerate(img_paths)):
        run.__wrapped__(idx, ip)
    t1 = time.time()
    print("time:", t1 - t0)
```

这里有一个小的Python知识点：可以通过`func.__wrapped__` 属性来获取 _被装饰的函数_ 对应的原始函数。

输出结果如下：
```bash
[>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>] 4000/4000, 316.3 task/s, elapsed: 13s, ETA:     0s
4000it [00:45, 88.84it/s]
time: 45.0268120765686
```
可以看到耗时从45ms下降到13ms，加速比3.4倍。

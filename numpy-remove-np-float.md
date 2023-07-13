---
title: 关于 np.float 被删除的问题
date: 2023-02-05 11:34:04
tags:
- Python
- Numpy
---
### 1. 概述
在Numpy 1.24版本中，[删除](https://numpy.org/doc/stable/release/1.24.0-notes.html#expired-deprecations)了像`np.float`、`np.int` 这样的 Python 内置类型的 alias，因此以后在代码中使用这些类型会报错`AttributeError: module 'numpy' has no attribute 'float'`, 涉及的类型包括：
+ `numpy.bool`
+ `numpy.int`
+ `numpy.float`
+ `numpy.complex`
+ `numpy.object`
+ `numpy.str`
+ `numpy.long`
+ `numpy.unicode`

那该怎么解决这个错误呢？

TL;DR
+ 对于在标量上的操作，直接使用Python内置类型替换
```python
foo = np.random.rand(10)
# 原先用法，注意foo[0]是一个标量
bar = np.float(foo[0])
# 新用法
bar = float(foo[0])
```
+ 对于在`np.ndarray` 上的操作，使用`np.float64` 或`np.float32` 来替代，具体选择哪个需要自己根据情况来确定，不同类型精度会有不同，下面举两个例子:
```python
# 原先用法
foo = np.random.rand(10, dtype=np.float)
# 新用法
foo = np.random.rand(10, dtype=np.float32)

# 原先用法
foo = np.random.rand(10).astype(np.float)
# 新用法
foo = np.random.rand(10).astype(np.float32)
```

这里列出来了删除类型在标量和`np.ndarray` 上的替代，方便查找

| 原先类型       | 标量替换类型 | `np.ndarray`替换类型  |
| ---------- | ------------ | --------------------- |
| np.int     | int          | np.int32/np.int64     |
| np.float   | float        | np.float32/np.float64 |
| np.bool    | bool         | np.bool_              |
| np.complex | complex      | np.complex128         |
| np.object  | object       | -                     |
| np.str     | str          | np.str_               |
| np.long    | int          | np.int32/np.int64     |
| np.unicode | str          | np.str_               |

详细说明参考[NumPy 1.20.0 Release Notes](https://numpy.org/doc/stable/release/1.20.0-notes.html#deprecations)。

下面详细说说事情的来龙去脉。

<!--more-->
### 2. 代码验证

下面我搭建 Numpy 1.20.0 和 1.24.0 的环境进行简单测试，以及分析为什么会弃用这些类型。

首先是 Numpy 1.20.0 环境搭建与简单测试：
```bash
python -m venv np1.20
source np1.20/bin/activate
pip install numpy==1.20
python -c "import numpy as np; a = np.array([1.0], dtype=np.float)"
```
输出如下：
```bash
<string>:1: DeprecationWarning: `np.float` is a deprecated alias for the builtin `float`. To silence this warning, use `float` by itself. Doing this will not modify any behavior and is safe. If you specifically wanted the numpy scalar type, use `np.float64` here.
Deprecated in NumPy 1.20; for more details and guidance: https://numpy.org/devdocs/release/1.20.0-notes.html#deprecations
```

仔细看这段输出的话，可以发现从 Numpy 1.20 版本开始，Numpy已经弃用`np.float` 类型了，并且给出了替换建议，以及详细的说明文档[地址](https://numpy.org/devdocs/release/1.20.0-notes.html#deprecations)。

而在 Numpy 1.24版本里面，正式删除了`np.float`，可以用下面的代码来测试。
首先我们创建一个新的环境，安装Numpy 1.24版本，然后创建一个`np.float`类型的数组：
```bash
python -m venv np1.24
source np1.24/bin/activate
pip install numpy==1.24
python -c "import numpy as np; a = np.array([1.0], dtype=np.float)"
```

输出如下:
```bash
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/Users/name/np1.24/lib/python3.9/site-packages/numpy/__init__.py", line 284, in __getattr__
    raise AttributeError("module {!r} has no attribute "
AttributeError: module 'numpy' has no attribute 'float'
```
直接就报了我们开头提到的属性错误。

### 3. Why 
其实早在2015年，Numpy 开发者就在[策划](https://github.com/numpy/numpy/pull/6103)删除这些类型了，只不过当时使用范围太广，删除造成的影响太大，所以在近8年，1.20-1.24 4个版本的Warning后，才正式删除。
为什么要删除这些操作呢？我自己觉得是因为`np.float` 这种类型太容易误用了。大家都以为`np.float`是一个Numpy的数据类型，是`np.float32`的alias，但实际它是内置类型，是`int`类型的alias。
就像下面这个例子：
```python
>>> foo = np.array([10], dtype=np.int32)
>>> bar = np.int(foo)
>>> type(bar)
<class 'int'>
>>> baz = np.int32(foo)
>>> type(baz)
<class 'numpy.ndarray'>
```
可以看到，对`np.ndarray` 数组进行`np.int` 和`np.int32`的操作，一个得到`int`类型的变量，另一个得到的是`np.ndarray`类型的变量。

详细的原因可以参考上面的 issue 链接。


那最早为什么还要引入`np.float`呢？直接用Python内置的类型不好吗？其实这是在很早的Numpy版本中错误地引入的，那个版本`np.float`的含义就是`np.float64` ，只不过后来版本中`np.float` 的含义修改了，但如果直接删除`np.float`，有人使用老版本的Numpy，就会在执行`from numpy import *` 报错。当前那个老版本已经很少有人用了 ，所以就删除了。


### 4. 带来的影响
这个改动带来的影响可以说是非常大了，简单来说，在 Numpy 1.24.0以上的版本中，使用`np.float`的代码都会直接报错。而 Numpy 作为 Python 在科学计算中的基础包，被广泛使用的程度无需我赘述。
简单在GitHub 搜索了一下，光涉及到`np.float`的([结果1](https://github.com/search?q=np.float%29++lang%3APython++&ref=opensearch&type=code)， [结果2](https://github.com/search?q=np.float%28+lang%3APython++&ref=opensearch&type=code)）就有近9万行代码，我自己短期内就在两个仓库中遇到这个问题。好在解决办法也比较直接，希望可以顺利的过渡过去。



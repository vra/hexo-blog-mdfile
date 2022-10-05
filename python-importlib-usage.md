---
title: python importlib用法小结
date: 2022-05-28 07:31:46
tags:
- Python
- 总结
---
在使用Python的时候，大部分时候引入包，都是通过`import` 语句，比如`import numpy as np`。有时候为了更复杂的需求，我们需要用**程序化**的方式来引入包 (Programmatic Importing), 比如根据输入不同，选择执行两个不同包里面的同名函数，这时候就需要用到`importlib`这个库了。这里先从一个简单例子开始，逐渐深入地讲一下这个库的用法。
<!--more-->

## import_module用法
`importlib`是Python3.1增加的系统库，其中最常用的函数是其中的`import_module`，功能是用程序语句的方式替代`import`语句，用法如下：
```python
import importlib

# 与 import time 效果一样
time = importlib.import_module('time')
print(time.time())

# 与 import os.path as path 效果一样
path = importlib.import_module('os.path')
path.join('a', 'b')  # results: 'a/b'

# 相对引入, 一级目录，与 import os.path as path 效果一样
path = importlib.import_module('.path', package='os')
path.join('a', 'b')  # results: 'a/b'

# 相对引入，二级目录，与 import os.path as path 效果一样
path = importlib.import_module('..path', package='os.time')
path.join('a', 'b')  # results: 'a/b'
```

注意最后的例子中，相对引入时需要在前面增加`.`或者`..`来表示相对目录，如果直接使用`importlib.import_module('path', package='os')`会报错。

如果光看这几个例子的话，貌似跟`import` 没什么区别，而且语句变得更复杂了，有点多此一举的感觉。

其实不是的，**个人认为，`importlib`的强大之处是将`import`语句中写死的字面值改成了`import_module`函数中的参数，因此可以通过修改参数在外部用变量来控制实际import的包或者模块，大大地增加了灵活性。** 下面会举一个稍微实用一些的例子。

## 一个实际例子
假设我们在设计一个深度学习工具库，里面包含了N个网络模型（ResNet50, HRNet, MobileNet等等），每个模型的实现都有一个`load_model`的函数。由于计算设备的性能不同，需要调用的网络结构也会变化，我们需要根据外部传入的参数来判断实际load哪一个模型。

虽然采用`import`语句+`if-else`判断也能完成这个需求，举例实现如下:
```python
def run(model_name, input):
    if model_name == 'resnet_50':
        from resnet_50.model import load_model
    elif model_name == 'hrnet':
        from hrnet.model import load_model
    elif model_name == 'moblienet':
        from mobilenet.model import load_model

    model = load_model()
    output = model(input)
    return output
```
这种写法存在下面的两个问题：
 1. 写法很冗余, N个模型的话需要添加2N条语句
 2. 新增模型时需要修改调用处的代码，添加对应的import语句，不符合模块化的要求。

这时候采用`importlib`就能比较简洁地解决这个问题:
```python
import importlib


def run(model_name, input):
    load_model = importlib.import_module('load_model', package='{}.model'.format(model_name))

    model = load_model()
    output = model(input)
    return output
```
可以看到在这种场景下`importlib`确实能大大简化代码。

了解这些内容，日常使用这个库就没什么问题了（好像`importlib`针对普通用户场景的函数貌似就只有`import_module`这一个），别的一些进阶的概念~~在下个部分展开说一下~~(由于不太懂，暂时不展开了)。

## 参考
1. <https://docs.python.org/3/library/importlib.html>







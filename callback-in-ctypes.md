---
title: 在ctypes的C共享库中调用Python函数
date: 2022-10-04 11:42:22
tags:
- Python
- C++
- Numpy
---
### 1. 概述
[ctypes](https://docs.python.org/3/library/ctypes.html) 是Python标准库中提供的外部函数库，可以用来在Python中调用动态链接库或者共享库中的函数，比如将使用大量循环的代码写在C语言中来进行提速，因为Python代码循环实在是太慢了。大致流程是通过 ctypes 来调用C函数，先将Python类型的对象转换为C的类型，在C函数中做完计算，返回结果到Python中。这个过程相对是比较容易的。

现在有个更复杂的情况，我想要在C代码中调用Python中的某些函数来完成C代码的计算，比如在C代码的sort函数中，采用Python中定义的函数来进行大小判断。这个在Python中定义的函数在 ctypes 中称为回调函数 (callback function)。也就是说需要把Python函数当作变量传给C语言，想想还是有些难度。 但调查以后发现 ctypes 提供了 `CFUNCTYPE`来方便地进行回调函数定义，而C语言本身也是支持函数指针的，因此这个功能实现还算简单，具体展开如下。
<!--more-->

### 2. 一个最简单例子
先从最简单例子开始，跑通整体流程。假设我们有个回调函数，判断int类型的输入是不是大于0，那么可以在C语言这么写:
```c
// my_lib.c
int foo(int (*function_ptr)(int) , int a) {
	return function_ptr(a);
}
```
这个文件内容很简单，我们定义了一个C函数`foo`，它调用Python传过来的回调函数，直接返回结果。

这里使用了C语言的函数指针类型，`int (function_ptr)(int)`中函数指针变量名是`function_ptr`, 返回值类型是前面的int，参数类型是后面的int。

我们在C语言里面只是简单地调用了Python传过来的函数指针，并直接将结果返回，实际使用时其实是需要在Python函数算完后，利用输出进行更多操作，否则直接在Python里面计算函数就可以了，没必要传函数到C，算法结果再返回给Python。

使用下面的命令来将上述C文件编程成共享库`my_lib.so`:
```bash
gcc -shared -o my_lib.so my_lib.c
```
这个命令会在当前目录下会生成`my_lib.so`。

然后在Python文件中定义这个回调函数的具体实现，以及调用共享库`my_lib.so`中定义的`foo`函数：
```python
# file name: ctype_callback_demo.py
import ctypes as c
from ctypes import cdll


# 定义回调函数
@c.CFUNCTYPE(c.c_int, c.c_int)
def callback_func(a):
    res = int(a > 0)
    return res


if __name__ == '__main__':
    a = 2

	# 载入共享库
    lib = cdll.LoadLibrary('./my_lib.so')

	# 调用共享库中的foo函数
    res = lib.foo(callback_func, a)

    print('{} > 0 = {}'.format(a, res))
```
所有 magic 的事情都被 ctypes 这个库给做了，留给我们的都是比较简单的接口。

`@c.CFUNCTYPE` 这个装饰器就是用来声明回调函数的，装饰器的第一个参数是函数的返回类型，第二个参数开始，就是回调函数自己的参数的类型。如果回调函数没有返回值，那`@c.CFUNCTYPE`后面的第一个参数设置为`None`。

然后执行这个Python脚本，可以得到下面的输出:
```bash
$ python ctype_callback_demo.py
2 > 0 = 1
```

### 3. Numpy.ndarray 类型的参数如何使用

ctypes 对 Python原生类型支持是没问题的，但我们还会经常用到Numpy的ndarray对象，它们该如何转换为C语言可以识别的类型呢？因为跨语言的类型转换不对的话，结果就会有问题。

Numpy 提供了 numpy.ndarray.ctypes 属性，可以来完成这个操作。

比如C文件中，需要一个float 指针类型的输入:
```cpp
// my_lib.c
int foo(int (*function_ptr)(float*) , float* a) {
	return function_ptr(a);
}
```

我们需要将Numpy.ndarray对象进行转换，传给C函数:

```python
import ctypes
import numpy as np


# 获取C的float指针类型
c_float_p = ctypes.POINTER(ctypes.c_float)

data = np.random.rand(3, 3).astype(np.float32)

# 将np.ndarray 对象的类型转换为C的float指针类型
data_p = data.ctypes.data_as(c_float_p)

# 调用共享库中的foo函数
my_lib.foo(data_p)
```

### 参考
1. <https://docs.python.org/3/library/ctypes.html#callback-functions>
2. <https://stackoverflow.com/questions/3195660/how-to-use-numpy-array-with-ctypes>

---
title: Numpy set_printoptions函数用法
date: 2023-09-30 12:57:30
tags:
 - Python
 - Numpy
---

Numpy是Python中常用的数值计算库，我们经常需要用到Numpy来打印数值，查看结果。为了能精确地控制Numpy打印的信息，Numpy提供了`set_printoptions` [函数](https://numpy.org/doc/stable/reference/generated/numpy.set_printoptions.html)，包含数个参数，能满足数值打印的需要。

这里以iPython中操作作为示例，从浅入深，一步步地探索set_printoptions提供的功能，如何来满足我们的打印需求。
<!--more-->

### precision
首先用Numpy创建一个float64 类型的np.ndarray，并打印数值：
```python
In [1]: import numpy as np

In [2]: a = np.random.rand(3)

In [3]: print(a)
[0.63039295 0.09185505 0.02203224]
```
可以看到输出的float数组保留了8位小数位，这是因为Numpy默认的设置是显示8位小数位。
如果只想显示2位小数位，该怎么操作呢？
这时候就需要用到`set_printoptions`的`precsion`的选项了，它就是用来控制显示的小数位：
```python
In [4]: np.set_printoptions(precision=4)

In [5]: print(a)
[0.6304 0.0919 0.022 ]
```

可以看到通过设置precsion=4，显示的数组输出保留4位小数。

⚠️需要注意的是，这个设置对float类型的数值无效：
```python
In [14]: a = np.random.rand()

In [15]: type(a)
Out[15]: float

In [16]: print(a)
0.40944018143470295

In [17]: np.set_printoptions(precision=4)

In [18]: print(a)
0.40944018143470295
```

所以使用时注意类型是`np.ndarray`还是`float`。

### suppress
假设我们需要获取一组很小的数值，并且需要显示结果：
```python
In [2]: a = np.random.rand(3) * 1e-5

In [3]: print(a)
[9.49522547e-06 4.55101001e-06 4.01284118e-06]

```
可以看到打印时用了科学计数法。
有没有办法不使用科学计数法呢，`set_printoptions`提供了`suppress`参数，将其设置为`True`，就会禁用科学计数法：

```python
In [4]: np.set_printoptions(suppress=True)

In [5]: print(a)
[0.0000095  0.00000455 0.00000401]
```


`suppress` 参数有一个例外情况，就是对于整数部分大于8位的，即使设置`suppress=True` ，仍然会显示科学计数法：
```python
In [8]: a = np.random.rand(3) * 1e10

In [9]: print(a)
[9.35772525e+09 4.14513333e+09 4.59176775e+09]

In [10]: a = np.random.rand(3) * 1e8

In [11]: print(a)
[37984517.91633694 87748330.34519586 21101693.42416701]

In [12]: a = np.random.rand(3) * 1e9

In [13]: print(a)
[4.46826342e+08 5.17327105e+08 9.07218130e+08]
```

那有没有办法解决这个问题呢？这里就需要用到`set_printoptions` 提供的另一个参数`formatter`。

### formatter
`formatter`接受一个dict类型的参数，其中dict的key表示参数的类型，而dict的value则是一个函数或者format字符串，表示如何对对应的类型进行打印。

举个简单的例子，我想在所有float类型的数组的每个元素后面加一个字母`f`:
```python
In [21]: a = np.random.rand(3)

In [22]: np.set_printoptions(precision=4, formatter={'float': lambda x: str(x) + 'f'})

In [23]: print(a)
[0.6925034861246904f 0.0613911477046164f 0.3348313234151774f]
```
`formatter`参数是个dict，key是“float"，表示对float类型的数组进行操作，value是一个lambda函数，将输入转换为str字符串再加一个f。

在这里也可以看到，np.float64数组中元素的实际长度是16位小数。默认显示的8位数值只是它的一个近似。

除了lamda函数外，也可以用Python的format格式函数来作为`formatter`参数dict的value：
```python
In [33]: a = np.random.rand(3)

In [34]: np.set_printoptions(formatter={'float': '{:.2f}'.format})

In [35]: print(a)
[0.70 0.91 0.82]
```
可以看到，这里可以用f-string和format函数中使用的语法格式，对于用惯f-string的小伙伴来说，以这种方式来控制显示格式简直太舒服了。

关于python format的语法，可以参考我之前写的[教程](https://zhuanlan.zhihu.com/p/632687543)。

另外需要注意，设置`formatter`后，会覆盖`precision`参数，也就是显示多少位数以`formatter`中设置为准:
```python
In [25]: a = np.random.rand(3)

In [26]: np.set_printoptions(precision=4, formatter={'float': lambda x: str(x) + 'f'})

In [27]: print(a)
[0.1604388367489663f 0.6047908263355061f 0.1645621828526913f]
```

根据Numpy 文档，`formatter`支持的类型包括下面这些：
- ‘bool’
- ‘int’
- ‘timedelta’ : a [`numpy.timedelta64`](https://numpy.org/doc/stable/reference/arrays.scalars.html#numpy.timedelta64 "numpy.timedelta64")
- ‘datetime’ : a [`numpy.datetime64`](https://numpy.org/doc/stable/reference/arrays.scalars.html#numpy.datetime64 "numpy.datetime64")
- ‘float’
- ‘longfloat’ : 128-bit floats
- ‘complexfloat’
- ‘longcomplexfloat’ : composed of two 128-bit floats
- ‘numpystr’ : types [`numpy.bytes_`](https://numpy.org/doc/stable/reference/arrays.scalars.html#numpy.bytes_ "numpy.bytes_") and [`numpy.str_`](https://numpy.org/doc/stable/reference/arrays.scalars.html#numpy.str_ "numpy.str_")
- ‘object’ : _np.object__ arrays
- ‘all’ : sets all types
- ‘int_kind’ : sets ‘int’
- ‘float_kind’ : sets ‘float’ and ‘longfloat’
- ‘complex_kind’ : sets ‘complexfloat’ and ‘longcomplexfloat’
- ‘str_kind’ : sets ‘numpystr

好了，说了这么多，那回到上面的问题，到底该怎么控制整数位大于8的float数组不用科学计数法呢？有了formatter参数，就很简单了：
```python
In [36]: a = np.random.rand(3) * 1e10

In [37]: np.set_printoptions(formatter={'float': '{:.8f}'.format})

In [38]: print(a)
[7694883457.28612423 864845466.08411431 6505022487.23314571]
```
使用format格式语言轻松完成。

有些时候，数组中的元素长度各不相同，打印时要么对不齐不好查看，要么自动转换为科学计数法也不好分析，利用`formatter`能够显示对齐的数值，大大方便了数据查看：
```python
In [1]: import numpy as np

In [2]: a = np.array(
   ...: [[1,  -1000, 2222222222.33333333],
   ...: [233, 240.03333333333333333333333333, 8.0],
   ...: [1.0, 2.0, 3.0]]
   ...: )

In [3]: print(a)
[[ 1.00000000e+00 -1.00000000e+03  2.22222222e+09]
 [ 2.33000000e+02  2.40033333e+02  8.00000000e+00]
 [ 1.00000000e+00  2.00000000e+00  3.00000000e+00]]

In [4]: np.set_printoptions(formatter={'float': '{:>20.8f}'.format})

In [5]: print(a)
[[          1.00000000       -1000.00000000  2222222222.33333349]
 [        233.00000000         240.03333333           8.00000000]
 [          1.00000000           2.00000000           3.00000000]]
```
这里利用了`>N`的format语法，向右对齐。

### threshold和edgeitems
假如我们有一个很大的数组(1024x4)，打印时默认只显示开始三行和最后三行：

```python
In [1]: import numpy as np

In [2]: a = np.random.rand(1024, 4)

In [3]: print(a)
[[0.5159347  0.06396333 0.18446106 0.06163127]
 [0.96894042 0.278889   0.25117021 0.9757328 ]
 [0.42980522 0.44724705 0.89322128 0.19697129]
 ...
 [0.31956847 0.4790065  0.45595315 0.98816687]
 [0.35240443 0.44400784 0.76815952 0.18499155]
 [0.33888548 0.50811964 0.32341108 0.98617324]]
```
这是因为Numpy默认设置，当数组的元素个数大于1000时，就会只显示开头和结尾部分。


如果想多显示一些数据，看更多内容，该怎么操作呢？
`set_printoptions`提供了`threshold`参数，用于控制多少个元素后显示部分，另一个参数`edgeitems`,控制显示缩略部分的行数。

因此可以修改这两个参数，修改显示效果：
```python
In [4]: np.set_printoptions(edgeitems=5)

In [5]: print(a)
[[0.5159347  0.06396333 0.18446106 0.06163127]
 [0.96894042 0.278889   0.25117021 0.9757328 ]
 [0.42980522 0.44724705 0.89322128 0.19697129]
 [0.41831831 0.32864348 0.9599147  0.04244498]
 [0.17307071 0.70541496 0.12485861 0.68987846]
 ...
 [0.36880553 0.66404444 0.12623872 0.32754608]
 [0.53076768 0.76770867 0.36680954 0.58596153]
 [0.31956847 0.4790065  0.45595315 0.98816687]
 [0.35240443 0.44400784 0.76815952 0.18499155]
 [0.33888548 0.50811964 0.32341108 0.98617324]]
```
### linewidth
linewidth参数用来控制一行显示多少个字符，默认是75，通过修改这个参数，能在一行显示更多数据：
```python
n [3]: import numpy as np

In [4]: a = np.random.rand(1024,6)

In [5]: np.set_printoptions(precision=16)

In [6]: print(a)
[[0.6151590922948798 0.8394381715187383 0.1287492144726177
  0.432486748198503  0.008210600687992  0.5251777687645207]
 [0.8986836534319551 0.5275521098334796 0.1275787604074625
  0.2088067024068581 0.9728215202746345 0.0222310180458779]
 [0.1919751621010076 0.7593251629630882 0.2216025287318845
  0.1693395870716256 0.0447174013709218 0.2669167788671162]
 ...
 [0.2056367250351134 0.1961953658298233 0.6844119224272207
  0.396808314963211  0.2270659358855954 0.1694468143457141]
 [0.0404784779577213 0.977932794679906  0.319154876583544
  0.6301954893143036 0.4533581710958777 0.4980767389069806]
 [0.5722796781670568 0.8683487818109435 0.819417328117305
  0.5286251921005498 0.2252964609019765 0.7439441509500194]]

In [7]: np.set_printoptions(linewidth=150)

In [8]: print(a)
[[0.6151590922948798 0.8394381715187383 0.1287492144726177 0.432486748198503  0.008210600687992  0.5251777687645207]
 [0.8986836534319551 0.5275521098334796 0.1275787604074625 0.2088067024068581 0.9728215202746345 0.0222310180458779]
 [0.1919751621010076 0.7593251629630882 0.2216025287318845 0.1693395870716256 0.0447174013709218 0.2669167788671162]
 ...
 [0.2056367250351134 0.1961953658298233 0.6844119224272207 0.396808314963211  0.2270659358855954 0.1694468143457141]
 [0.0404784779577213 0.977932794679906  0.319154876583544  0.6301954893143036 0.4533581710958777 0.4980767389069806]
 [0.5722796781670568 0.8683487818109435 0.819417328117305  0.5286251921005498 0.2252964609019765 0.7439441509500194]]
```
可以看到，增加linewidth到150后，以前一行显示不了的数据现在可以在一行上显示了。

### nanstr和infstr
nanstr和infstr参数用来控制nan和inf数值的显示字符，默认是`nan`和`inf`，如果好奇想修改的话，可以设置对应的参数：
```python
In [12]: a = np.array([np.nan, np.inf])

In [13]: print(a)
[nan inf]

In [14]: np.set_printoptions(nanstr='非数', infstr='∞')

In [15]: print(a)
[非数  ∞]
```
有点好玩，但建议别修改，不然别人不知道你在do what 🤷

### sign
sign参数用来控制每个数字前显示的符号，默认是`-`,也就是只有负数前面显示减号。如果是`+`，则在正数前面添加加号。如果是` `,则在正数前面添加空格：
```python
In [1]: import numpy as np

In [2]: a = np.random.rand(3) - 0.5

In [3]: print(a)
[ 0.44889495 -0.25608263 -0.23228835]

In [4]: np.set_printoptions(sign='+')

In [5]: print(a)
[+0.44889495 -0.25608263 -0.23228835]

In [7]: np.set_printoptions(sign=' ')

In [8]: print(a)
[ 0.44889495 -0.25608263 -0.23228835]
```

### 恢复默认配置
如何恢复默认配置呢？可以这么设置：
```python
np.set_printoptions(edgeitems=3, infstr='inf',linewidth=75, nanstr='nan', precision=8,suppress=False, threshold=1000, formatter=None)
```

### 使用with语句
通过使用with语句，可以临时修改打印配置项，在退出with语句的时候恢复默认配置，这样也减少侵入式地修改，避免造成不必要的后果。
在使用with语句时，需要将`np.set_printoptions` 替换为`np.printoptions`，也就是去掉函数中的`set_`前缀，函数的所有参数都一样。使用例子：
```python
In [6]: a = np.random.rand(3)

In [7]: with np.printoptions(formatter={'float': lambda x: str(x)+'f'}):
   ...:     print(a)
   ...:
[0.24752544521208586f 0.01852100917834376f 0.9358432384604951f]

In [8]: print(a)
[0.24752544521208586f 0.01852100917834376f 0.9358432384604951f]
```

这就是`set_printoptions`几乎全部的参数了，更多用法，参考官方[文档](https://numpy.org/doc/stable/reference/generated/numpy.set_printoptions.html)。

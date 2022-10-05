---
title: python的列表推导式和生成器表达式对比
date: 2022-05-22 13:16:58
tags:
- Python
---
## 概述
Python中的列表推倒式(List Comprehension) 和 生成器表达式（Generator Expression）是两种很相似的表达式，但含义却不大不同，这里做一个对比。
<!--more-->
## 列表推导式
列表推导式是比较常用的技术，能将本来需要`for` loop和`if else`语句的情况简化成一条指令，最终得到一个列表对象:
```python
even = [e for e in range(10) if e % 2 == 0]
```
具体细节不过多展开，相信很多使用Python的人都已经足够了解这种语法了。

需要注意的一点是，列表推导式不是惰性计算 ( Lazy Loading) 的，因此所有的列表成员都在声明完语句后立即计算 (Eager Loading)，因此在数组成员很多的情况下，速度会很慢，例如下面的在IPython环境里面的三个列表推导式的耗时统计:
```python
In [1]: %timeit even = [e for e in range(100000) if e % 2 == 0]
5.5 ms ± 24.8 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)

In [2]: %timeit even = [e for e in range(1000000) if e % 2 == 0]
58.9 ms ± 440 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)

In [3]: %timeit even = [e for e in range(100000000) if e % 2 == 0]
5.65 s ± 26.5 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```
可以看到随着元素个数的增加，列表推导式执行的时间也相应变长，占用的内存也会变大。

有一种情况是，我们定义了很多很多的数组元素，但是最后并不是所有的元素都能用到，例如经过几条命令，最后可能只有列表里面的前10个元素会用到，或者只有符合某些条件的元素会用到，这样的话，Eager模式就白白花费了时间，白白花费了内存来创建很多用不到的元素，这显然有很大的改进空间。

## 生成器表达式
生成器能表达式解决上面的问题，它的元素迭代是惰性的，因此只有需要的时候才生产出来，避免了额外的内存开销和时间开销: 生成器表达式不管元素数目多大，创建时都是常数时间，因为它并没有立即创建元素。

那么生成器表达式的语法是怎么样的呢，很简单，只需要把列表推导式中的方括号改为圆括号:
```python
even_gen = (e for e in range(10) if e % 2 == 0)
```
注意它的类型是生成器类型:
```
type(even_gen)
# generator
```
创建生成器表达式的耗时统计:
```bash
In [1]: %timeit even_gen = (e for e in range(100000) if e % 2 == 0)
376 ns ± 2.61 ns per loop (mean ± std. dev. of 7 runs, 1,000,000 loops each)

In [2]: %timeit even_gen = (e for e in range(10000000) if e % 2 == 0)
382 ns ± 1.63 ns per loop (mean ± std. dev. of 7 runs, 1,000,000 loops each)

In [3]: %timeit even_gen = (e for e in range(1000000000) if e % 2 == 0)
384 ns ± 2.85 ns per loop (mean ± std. dev. of 7 runs, 1,000,000 loops each)
```
可以看到随着元素的增加，创建时间基本不变，而且比列表推导式的耗时要低不少。


## 使用场景选择
那么是不是就是说使用中可以用生成器表达式替代列表推导式了呢，也不尽然，因为列表推导式得到的是一个列表，很多便捷操作（如slice等）可以作用到上面，而生成器表达式则不行：
```python
In [17]: even = [e for e in range(10) if e % 2 == 0]

In [18]: even[:3]
Out[18]: [0, 2, 4]

In [19]: even_gen = (e for e in range(10) if e % 2 == 0)

In [20]: even_gen[:3]
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
Input In [20], in <cell line: 1>()
----> 1 even_gen[:3]

TypeError: 'generator' object is not subscriptable
```
而且两者有一个致命的区别：生成器表达式只能迭代一次，而列表推导式可以使用很多次，举例如下:
```python
In [22]: even_gen = (e for e in range(10) if e % 2 == 0)

In [23]: for e in even_gen:
    ...:     print(e)
    ...:
0
2
4
6
8

In [24]: for e in even_gen:
    ...:     print(e)
    ...:

```
可以看到生成器表达式在第二次迭代的时候，里面已经没有元素了！即第一次迭代已经全部生成出来了，而列表推导式是每次迭代都是有相同的内容:
```
In [25]: even = [e for e in range(10) if e % 2 == 0]

In [26]: for e in even:
    ...:     print(e)
    ...:
0
2
4
6
8

In [27]: for e in even:
    ...:     print(e)
    ...:
0
2
4
6
8
```

因此总结来说，
+ 如果要多次迭代时，建议使用列表推导式
+ 如果数组很大或者有无穷个元素，建议使用生成器表达式
+ 其他场景：两者均可，自己看情况使用一个，如果没有速度和方便度的问题即可，如果有问题换另一个再试试

## 参考
1. <https://stackoverflow.com/questions/47789/generator-expressions-vs-list-comprehensions>
2. <https://docs.python.org/3/howto/functional.html#generator-expressions-and-list-comprehensions>

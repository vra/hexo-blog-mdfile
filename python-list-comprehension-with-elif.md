---
title: Python转换elif语句为列表推导式
date: 2022-06-28 22:32:11
tags:
- Python
- 总结
---

## 1. 概述
今天才发现，在Python的列表推导式里面，也可以使用多个else，也就是elif的情况，具体来说，可以将下面的一长串的elif 语句转换成一句列表推导式，大大简化代码:
```python
if cond1:
	do1
elif cond2:
	do2
elif cond3:
	do3
else:
	do4
```
转换成列表推导式如下：
```python
res = [do1 if cond1 else do2 if cond2 else do3 if cond3 else do4][0]
```
Python喜爱值+1，代码行数-N。
<!--more-->

## 2. 几个例子
原先代码：
```python
if a > 10:
	return 'large'
elif a > 5:
	return 'middle'
else:
	return 'small'
```
可以转换为下面的形式：
```python
res = ['large' if a > 10 else 'middle' if a > 5 else 'small']
```

任意多个elif都是可以的，下面的代码验证了两种写法结果是一致的:
```python
import numpy as np


def func1(a):
    if a > 0.9:
        return 'a'
    elif a > 0.7:
        return 'b'
    elif a > 0.5:
        return 'c'
    elif a > 0.3:
        return 'd'
    elif a >= 0.1:
        return 'e'


def func2(a):
    return ['a' if a > 0.9 else 'b' if a > 0.7 else 'c' if a > 0.5 else 'd' if a > 0.3 else 'e'][0]


if __name__ == '__main__':
    arr = np.random.rand(10)
    for a in arr:
        print(func1(a) == func2(a))
```

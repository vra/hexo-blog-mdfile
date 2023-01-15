---
title: 使用 defaultdict 来简化 dict 的初始化
date: 2022-12-10 15:17:17
tags:
- Python
- 总结
---

## 1. 概述
在我们使用Python中的dict时，常常需要判断某个关键字是否已经在dict中，如果不存在则创建，非空则进行另外的操作。例如统计一篇文章中所有单词出现次数的代码，大致写法如下：
```python
words_num = {}

for word in words:
    if word not in words_num.keys():
        words_num[word] = 1
    else:
        words_num[word] += 1
```
这样写总是需要判断key是否在dict中，不是很优雅。

Python标准库collections中[defaultdict](https://docs.python.org/3/library/collections.html#defaultdict-objects)类可以很好的解决这个问题。这个类使用与dict几乎一样，除了可以在初始化时设置key的默认类型和数值。类的声明格式为`defaultdict(default_factory=None, /[, ...])`，`default_factory`是一个`callable`的变量。

别的使用与dict无异，正常使用即可。
<!--more-->

例如，`foo = defaultdict(int)`表示foo中的key的默认类型是int，且默认值为int的默认值0，我们可以获取**任意**的key，不需要手动初始化key:
```python
>>> from collections import defaultdict
>>> foo = defaultdict(int)
>>> foo['a']
0
>>> foo['b']
0
>>> foo['whatever']
0
>>> foo['a'] += 1
>>> foo['a']
1
```
所以最开始的例子可以简化为如下:
```python
from collections import defaultdict


words_num = defaultdict(int)

for word in words:
    words_num[word] += 1
```
可以看到使用defaultdict后，代码中只需要关注上层逻辑（统计单词的出现次数），而不需要关注具体的语法的代码实现（dict是否存在某个key，没有的话xxx，有的话xxx），因此世界变得更美好了一些。

除了int外，用list，tuple，dict，set等作为变量也比较常见。除了内置类型外，还可以自定义函数，比如设置key的默认值为`'China'`:
```python
>>> from collections import defaultdict
>>> def set_default_contry():
...     return "China"
...
>>> person_from = defaultdict(set_default_contry)
>>> person_from['张三']
'China'
>>> person_from['李四']
'China'
>>> person_from['Tim'] = 'USA'
>>> person_from
defaultdict(<function set_default_contry at 0x10896eca0>, {'a': 'China', '张三': 'China', '李四': 'China', 'Tim': 'USA'})
```

`defauldict`是一个简单但很好用的功能，在日常的使用中还是能减少一些代码复杂度的。希望这篇小文能给让你写代码更容易，更开心。
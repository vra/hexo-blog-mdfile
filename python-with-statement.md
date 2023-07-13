---
title: python 多个with 语句一起使用
date: 2023-01-20 13:37:24
tags:
- Python
- 总结
- TIL
---
在读《流畅的Python》时，偶然看到下面的语句：
```python
with urlopen(URL) as remote, open(JSON, 'wb') as local:
    local.write(remote.read())
```
突然才发现，原来多个with语句可以写到一起!
<!--more-->

我之前都是每个`with`一个层级，像下面这样：
```python
with open('in_file') as f:
    with open('out_file' 'w') as of:
        for line in f:
            of.write(line)
            ...
```
这样写每个with语句需要缩进一次，阅读起来逻辑不连续，而且很容易超过每行的字符限制，导致需要换行等问题，不是很方便。

经过这个偶然的发现，以后上面的代码可以这样写了：
```python
with open('in_file') as f, open('out_file' 'w') as of:
    for line in f:
        of.write(line)
        ...
```

同时看 `with` 语句的[官方文档](https://docs.python.org/3/reference/compound_stmts.html#the-with-statement)，发现从Python 3.10版本起，还可以用括号将多个with语句括起来:
```python
with (
    open("face_model_choice.txt") as f,
    open("ttt.txt", "w") as of1,
    open("ttt2.txt", "w") as of2,
):
    for line in f:
        of1.write(line)
        of2.write(line)
		...
```
这样看起来也更简洁了。


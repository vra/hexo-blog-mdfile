---
title: doctest 用法简介
date: 2021-02-02 22:43:43
tags:
 - Python
 - Unit Test
---
## 概述
[doctest](https://docs.python.org/3/library/doctest.html) 是 python 系统库中用于交互式会话例子测试的工具，用于搜索以 `>>>` 开头的语句，并且将其作为Python命令，对结果进行测试。

这个工具可以方便地用于检测自己写的库是否有bug，例如某些函数功能可能发生改变，借此工具可以方便地对代码中的示例语句进行测试。
<!--more-->
## 基本用法
假如我们有一个 Python 脚本 `foo.py`, 其中有一些 `>>>` 命令：
```python
# file name: foo.py
"""
My square function.
Usage:
>>> a = my_square(4)
>>> b = my_square(3)
>>> a + b
25
"""
def my_square(num):
    return num * num
```
为了测试我们的 docstring 中的示例用法（即以`>>>` 开头的命令）是否跟代码实现相符合，可以使用下面的命令来操作：
```bash
python3 -m doctest foo.py
```
没有报错的话默认是没有输出的，如果要看中间的执行信息，可以增加 `-v` 参数:
```bash
python3 -m doctest -v foo.py
```
另外针对只有运行命令记录，没有 python 语句的情况，可以把把命令记录保存到 `.txt` 文件中，然后使用同样的调用命令。例如把下面的内容保存到 `foo.txt` 文件中：
```text
>>> a, b = 2, 3
>>> a+b
5
```
那么就可以使用下面的命令调用
```bash
python3 -m doctest -v foo.txt
```

输出结果如下：
```bash
Trying:
    a, b = 2, 3
Expecting nothing
ok
Trying:
    a+b
Expecting:
    5
ok
1 items passed all tests:
   2 tests in foo.txt
2 tests in 1 items.
2 passed and 0 failed.
Test passed.
```

可以看到 `doctest` 会对文件中的每一行进行读取，然后计算期望的值和实际的值是否一样，如果不一样就会报错。例如我们尝试修改上面的 `foo.txt` 为下面的内容:
```text
>>> a, b = 2, 3
>>> a+b
6
```
即故意把2+3的结果修改为6，执行 `doctest` 命令，结果如下：
```bash
Trying:
    a, b = 2, 3
Expecting nothing
ok
Trying:
    a+b
Expecting:
    6
**********************************************************************
File "foo.txt", line 2, in foo.txt
Failed example:
    a+b
Expected:
    6
Got:
    5
**********************************************************************
1 items had failures:
   1 of   2 in foo.txt
2 tests in 1 items.
1 passed and 1 failed.
***Test Failed*** 1 failures.
```
可以看到，测试出错了，而且出错的详细信息也列出来了。

另一种使用的方法是在 python 脚本中增加 `doctest.testmod()` 函数调用，方法如下：
```python
# file-name: foo.py
"""
example usage:
>>> a, b = 2, 3
>>> a+b
5
"""
if __name__ == "__main__":
    import doctest
    doctest.testmod()
```
使用下面的命令来执行脚本:
```bash
python3 foo.py -v
```
输出结果如下：
```bash
Trying:
    a, b = 2, 3
Expecting nothing
ok
Trying:
    a+b
Expecting:
    5
ok
1 items passed all tests:
   2 tests in __main__
2 tests in 1 items.
2 passed and 0 failed.
Test passed.
```
对于 `.txt` 文件的测试，使用 `doctest.testfile()` 函数：
```python
import doctest
doctest.testfile("example.txt")
```

## 一些使用注意点
1. `>>>` 缩进多个层次对结果没有影响，`doctest` 测试之前会对每行前面的空格进行删除。
2. doctest 也可以对Error 进行测试，如果想要测试各种特殊case导致的错误的话，doctest是个不错的工具

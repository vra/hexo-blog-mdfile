---
title: Python 字符串的format用法
date: 2022-05-28 10:57:49
tags:
- Python
- 总结
---

## 1. 引入
我有一个朋友，某天突然问我：你知道下面的Python语句什么含义，结果是多少吗？
```python
'{:😄^+#20_x}'.format(12345)
```
我一看，十脸懵逼，吓得赶紧学了学Python的Format字符串的用法，总算明白了这个语句的含义。你想了解这个语句到底是什么鬼吗，欢迎跟我一起学。
<!--more-->

## 2. 整体说明
Python的Format语法，可以用在两个场景：一个是`{}`.format中，另一个是f-string中，`f{xxx}'中，只不过后者支持外部定义的变量:
```python
# .format way 1
print('Hello {}!'.format('World'))

# .format way 2
print('Hello {name}!'.format(name='World'))

# f-string
name = 'World'
print(f'Hello {name}!')
```

为了应对更复杂的使用场景，Python设计了一套全面的语法，来涵盖所有的使用情况。具体来说，这套语法将一个Format 语句分成五部分，分别是:
```bash
 "{" [字段名称部分] ["!" 格式转换部分] [":" 格式规范部分] "}"
 ```
 也就是左大括号和右大括号以及中间的核心三个部分, 其中方括号中的内容是可选的，也就是说最简单的format语法就是`{}`.format('xxx')，会打印format后的第一个内容。

 下面分开看看核心的三个部分。

 ## 3. 字段名称部分
 这一部分是用来定位要进行操作的变量。大括号中的编号对应这实际传入的参数，例如:
 采用关键字形式：
 ```python
 'My name is {name}, I am {age} years old'.format(name='Root', age=100)
 # My name is Root, I am 100 years old
 ```
 这里的`{name}`对应format后面的关键字形式的参数name。

 另一种是使用参数序号：

 ```python
 'My name is {0}, I am {1} years old'.format('Root', 100)
 # My name is Root, I am 100 years old
 ```
 这里的`{0}`对应`Root`, `{1}` 对应100，如果有更多的参数的话，编号按顺序往下继续。

 注意这里的`{idx}`在字符串中可以出现任意次，且出现的顺序是任意的：
 ```python
'{5} {5} {2}'.format('a','b','c','d','e', 'f')
# f f c
```

如果下标越界的话，会报错:
```python
'{5} {5} {2}'.format('a','b','c')
IndexError: Replacement index 5 out of range for positional args tuple
```

另外一个特性是，可以忽略括号中的编号，这时候就按照从0开始的顺序来读取输入：
```python
 # 下面的命令等效于 'My name is {0}, I am {1} years old'.format('Root', 100)
 'My name is {}, I am {} years old'.format('Root', 100)
 # My name is Root, I am 100 years old
 ```

 如果对复杂如列表或者字典，也可以使用下标或者属性来操作:
 ```python
 # 列表例子
friends = ['foo', 'bar']
'{0[0]}'.format(friends)
# 'foo'

# 字典例子
info = {'name': 'Root', 'age': 100}
'{0[name]}'.format(info)
# 'Root'

# 属性对象例子
from collections import namedtuple
Person = namedtuple('Person', ['name', 'age'])
p = Person('Root', 100)
'{0.name}'.format(p)
# 'Root
通过这些设置，能满足常见的需求。
```

## 4. 格式转换部分

这部分比较简单，在格式规范转换之前执行，通过感叹号加转换符号[r, s, a]之一，将原先的类型转换为字符串的类型，其中`!a` 表示对输入对象进行ascii()函数的调用，`!s`表示对输入对象进行str()函数的调用，而`!r`则调用repr()函数。

## 5. 格式规范部分
这部分是format格式中的大头，包含很多项设置，但都是可选的，例如上面的例子中我们都没有设置这部分。关于这部分的规范下面我们一一道来。
```plain
format_spec     ::=  [[fill]align][sign][#][0][width][grouping_option][.precision][type]
fill            ::=  <any character>
align           ::=  "<" | ">" | "=" | "^"
sign            ::=  "+" | "-" | " "
width           ::=  digit+
grouping_option ::=  "_" | ","
precision       ::=  digit+
type            ::=  "b" | "c" | "d" | "e" | "E" | "f" | "F" | "g" | "G" | "n" | "o" | "s" | "x" | "X" | "%"
```
### 5.1 fill 和align: 填充和对齐部分
这部分包括填充和对齐，填充部分是任意的一个字符，如'+', '*', 或者😄。不设置的话，默认使用空格来填充。对齐方式包括下面四种:
+ "<": 左对齐
```python
# 以空格左对齐，长度为10位
>>> '{0:<10}'.format(3.14)
'3.14      '
# 以星号左对齐，长度为10位
>>> '{0:*<10}'.format(3.14)
'3.14******'
```
+ ">"：右对齐
```python
# 以空格右对齐，长度为10位
>>> '{0:>10}'.format(3.14)
'      3.14'
```
+ "="：只对数值类型使用，表示对齐强制放到正负号和数值之间
```python
# 以星号数值对齐，长度为10位
>>> '{0:*=10}'.format(-3.14)
'-*****3.14'
```
+ "^": 居中对齐
```python
>>> '{0:*^10}'.format(-3.14)
'**-3.14***'
```
### 5.2 sign: 符号部分
这部分有三个选项；
```python
+ "+": 正负号都加符号
>>> '{0:+} {1:+}'.format(3.14, -3.14)
'+3.14 -3.14'
```
+ "-": 只有负数前面才加符号
```
>>> '{0:-} {1:-}'.format(3.14, -3.14)
'3.14 -3.14'
```
+ " ": 正数前面加空格，负数前面加负号
```python
>>> '{0: } {1: }'.format(3.14, -3.14)
' 3.14 -3.14'
```

### 5.3 #: 进制表示位
使用`#`号结合不同的进制表示符号(下面详细展开)，会在进制前面增加对应的负号，如二进制前增加`0b`, 八进制前增加`0o`, 十六进制前增加`0x`：
```python
# 二进制
>>> '{0:#b}'.format(233)
'0b11101001'
# 八进制
>>> '{0:#o}'.format(233)
'0o351'
# 十进制
>>> '{0:#d}'.format(233)
'233'
# 十六进制
>>> '{0:#x}'.format(233)
'0xe9
```

### 5.4 width: 显示的字符长度
这一部分表示显示多少位字符，包括pad的字符位。
```python
# 总长度为10位，不足的部分用默认的符号补齐
>>> '{0:10}'.format(233)
'       233'
```
### 5.5 grouping_option: 千位的标识符号
这部分表示千位的标识符号，有`,`和`\_`两种选择:
```python
>>> '{0:10,}'.format(23333333)
'23,333,333'
>>> '{0:10_}'.format(23333333)
'23_333_333'
```

### 5.6 .precision: 数值精度
这个表示浮点数的精度位数：
```python
>>> '{0:.3}'.format(3.14159)
'3.14'
```

### 5.7 type: 格式类型
这部分表示最终的展示类型，共有下面这些类:
```plain
"b" | "c" | "d" | "e" | "E" | "f" | "F" | "g" | "G" | "n" | "o" | "s" | "x" | "X" | "%"
```
每种的解释如下:
+ "b": 二进制表示
```python
>>> "{0:b}".format(8)
'1000
```
+ "c": 只支持整数，将其转换为对应的unicode符号
```python
>>> "{0:c}".format(23)
'\x17'
```
+ "d": 十进制表示 
```python
>>> "{0:d}".format(8)
'8'
```
+ "e": 科学计数法，采用小写的e
```python
>>> "{0:e}".format(8)
'8.000000e+00'
```
+ "E": 科学计数法，采用大写的E
```python
>>> "{0:E}".format(8)
'8.000000E+00
```
+ "f": 浮点数表示
```python
>>> "{0:f}".format(8)
'8.000000'
```
+ "F": 与"f"基本相同，除了将nan显示为NAN, inf显示为INF
```python
>>> "{0:F}".format(8)
'8.000000'
```
+ "g": 通用数据格式
```python
>>> "{0:g}".format(8)
'8'
```
+ "G": 通用数据格式
```python
>>> "{0:G}".format(8)
'8'
```
+ "n": 数值格式
```python
>>> "{0:n}".format(8)
'8'
```
+ "o": 八进制格式
```python
>>> "{0:o}".format(8)
'10
```
+ "s": 只能对字符串使用,字符串类型，默认输出类型，可以忽略
```python
>>> "{0:s}".format('www')
'www'
>>> "{0}".format('www')
'www'
```
+ "x": 十六进制
```python
>>> "{0:#x}".format(8)
'0x8'
```
+ "X": 十六进制，符号标识采用大写的X 
```python
>>> "{0:#X}".format(8)
'0X8'
```
+ "%": 只对数值类型使用，以百分比的形式显示
```python
>>> "{0:%}".format(8)
'800.000000%'
>>> "{0:%}".format(0.001)
'0.100000%'
```


## 6. 解答开头的神秘符号串
有了上面的知识，我们就可以解开文章开头的的神秘符号串了:
1. 以笑脸符号作为pad的字符，且居中对齐，总长为20个符号
2. 在正数前面增加加号
3. 显示为16进制，并且显示前面的进制标注符号
结果如下:

```python
>>>'{:😄^+#20_x}'.format(12345)
'😄😄😄😄😄😄+0x3039😄😄😄😄😄😄😄'
```

## 6. 参考
1. <https://docs.python.org/3/library/string.html>

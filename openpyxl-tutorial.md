---
title: openpyxl-读写Excel文件的Python库
date: 2019-02-27 22:27:35
tags:
 - Python
 - Linux
 - OpenPyxl
---
## 1. 概述
写脚本的时候，想要用Python读取Excel文件内容，谷歌搜索发现了openpyxl这个包，学习后发现简单地读写Excel文件还是比较方便的，库的设计也很简洁，没有太多深奥的东西。这里记录一下，说不定哪天还是会用到呢。
<!--more-->

## 2. 概念介绍
打开一个Excel文件的时候，首先我们会看到底部有“Sheet1”或“工作簿1”的文字，可见一个Excel文件是由一个或多个工作簿组成的。
每个工作簿的工作区，横向坐标是以字母为编号的，从A到Z;纵向是以数字为编号的，从1开始，一直往增大方向编号。由数字和字母为横纵坐标构成的每个小框叫做单元格，这是Excel的基本单位。字母和数字确定后，对应的单元格就唯一确定了；而单元格已知后，它对应的字母和数字也就确定了。
因此我们可以这样总结：
一个Excel文件由一或多个Sheet组成，而一个Sheet由字母和数字唯一表示的单元格们组成，这是一个三级的结构。下图表示一个名字为data.xlsx的Excel文件的3级层级结构。 
```bash
data.xlsx
├── Sheet1
│   ├── A1
│   ├── A2
│   ├── B1
│   └── B2
├── Sheet2
│   ├── A1
│   ├── A2
│   ├── B1
│   └── B2
└── Sheet3
    ├── A1
    ├── A2
    ├── B1
    └── B2
```
明白了这个结构，openpyxl的设计理念就很好理解了。
在opnepyxl里面，一个Excel文件对应着一个Workbook对象， 一个Sheet对应着一个Worksheet对象，而一个单元格对应着一个Cell对象，下面是一个最简单的例子，执行示例之前请使用`pip install --user openpyxl`安装openpyxl包。

```python
$ ipython3  # 在命令行打开ipython交互式界面
Python 3.5.2 (default, Nov 12 2018, 13:43:14) 
Type 'copyright', 'credits' or 'license' for more information
IPython 6.4.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: from openpyxl import load_workbook # load_workbook用于从一个xlsx文件读入数据

In [2]: wb = load_workbook('data.xlsx') # wb是data.xlsx对应的Workbook对象

In [3]: type(wb)
Out[3]: openpyxl.workbook.workbook.Workbook

In [4]: ws1 = wb['Sheet1'] # ws1是Sheet1对应的Worksheet对象

In [5]: type(ws1)
Out[5]: openpyxl.worksheet.worksheet.Worksheet

In [6]: cell1 = ws1['A1'] # cell1是Sheet1中第一个单元格对应的Cell对象

In [7]: type(cell1)
Out[7]: openpyxl.cell.cell.Cell

In [8]: cell1.value # 使用.value参数来获取Cell对象对应的值
Out[8]: 100

In [9]: cell1.value = 365 #使用.value参数来对单元格赋值

In [10]: cell1.value
Out[10]: 365
```
使用起来是不是很简单？这个例子看懂了，这篇总结80%的任务就完成了。下面是详细使用说明。

## 3. Workbook读写
如果要用openpyxl从头创建一个Excel文件，需要对Workbook进行默认初始化：
```python
>>> from openpyxl import Workbook
>>> wb = Workbook()
```
如果是要从现有Excel里面导入数据，使用load_workbook函数即可：
```python
>>> from openpyxl import load_workbook
>>> wb = load_workbook('data.xlsx')
```
要保存Workbook，调用Workbook的save函数就行：
```python
>>> wb.save('data.xlsx')
```

## 4. Sheet的操作
### 4.1. 获取Sheet对象
有下面几种方式可以得到Sheet对象：
 1. 调用Workbook对象的active属性，得到当前激活的工作簿：
 ```python
 >>> ws = wb.active
 >>> ws
 <Worksheet "Sheet">
 >>> ws.title
 'Sheet'
 ```
 2. 通过Workbook对象的[]函数，[]里面是Sheet对象的名字，所有工作簿的名字列表可以通过wb.sheetnames得到： 
 ```python
 >>> wb.sheetnames  ＃ sheetnames属性表示 Workbook对象包含的Sheet的名字
 ['Sheet']
 >>> ws=wb['Sheet']
 >>> ws
 <Worksheet "Sheet">
 ```
 3. 通过for循环迭代器得到:
 ```python
 >>> for ws in wb:
 ...     print(ws.title)
 ... 
 Sheet
 ```
 4. 从已有的工作薄复制过来：
 ```python
 >>> ws = wb.active
 >>> ws1 = wb.copy_worksheet(ws)
 >>> ws1
 <Worksheet "Sheet Copy">
 ```

### 4.2. Sheet对象属性
 Sheet对象有许多有用的函数和属性，基本的几个介绍如下。
 1. title，即工作薄的名称，显示在Excel底部
 ```python
 >>> ws.title
 'Sheet'
 ```
 2. parent，即所属的Ｗorkbook的名称
 ```python
 >>> wb1 = ws.parent
 >>> wb1 == wb
 True
 ```
 3. active_cell，即光标所在的单元格的编号
 ```python
 >>> ws.active_cell
 'B5'
 ```
 4. rows和columns，表示行和列的迭代器，通过for循环可以得到每行或每列的单元格元组
 ```python
 >>> for row in ws.rows:
 ...     print(row)
 ... 
 (<Cell 'Sheet'.A1>, <Cell 'Sheet'.B1>)
 (<Cell 'Sheet'.A2>, <Cell 'Sheet'.B2>)
 ```

## 5. Cell对象的操作
### 5.1. 获取对象
获取对象也有好几种方式，下面一一介绍。
 1. 通过工作簿对象的cell函数获取
 ```python
 >>> c = ws.cell(row=1, column=1)  # 获取第一行第一列的单元格
 >>> c.value  # 打印单元格的值
 '姓名'
 >>> c.value  = ‘Name’  # 重设单元格的值
 ```
 2. 通过工作薄对象的[]函数来获取，这里面获取方式比较灵活，举例如下：
 ```python
 >>> c = ws['A4']  # 获取第４行，第１列的单元格
 >>> c = ws['A']  # 获取第１列的所有单元格
 >>> c = ws['5']  # 获取第５行的所有单元格
 >>> c = ws['A1': 'B10']  # 获取第1行第1列到第10行第2列的矩形区域内的所有单元格
 >>> c = ws['A':'B']  # 获取第1列到第2列的所有单元格
 >>> c = ws[1:10]  # 获取第1行到第10行的所有单元格
 ```
 熟练使用这种操作，简单的任务就可以轻松处理了。
 3. 通过iter_cols或iter_rows来得到：
 ```python
 >>> for row in ws.iter_rows(min_col=1, max_col=2, min_row=1, max_row=3):
 ...     for c in row:
 ...             print(c.value)
 ... 
 姓名
 年龄
 ｗｗｗ
 24
 None
 None
 ```
 其中参数min_col和min_row是迭代时起始的列号和行号，max_col和max_row是结束的列号和行号，都是包含在迭代内部的。
 4. 通过工作簿对象的active_cell得到光标所在的单元格：
 ```python
 >>> ws.active_cell
 'B5'
 ```

## 6. 总结
上面介绍了openpyxl常见的用法，看完后你会发现有了这个库，用Python 操作Excel容易多了。这里是我最近用的一个例子：

更多用法请参考[官方教程](https://openpyxl.readthedocs.io)
下篇博客再见～

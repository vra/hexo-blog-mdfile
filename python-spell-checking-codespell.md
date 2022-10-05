---
title: 简单好用的英文拼写检查工具codespell
date: 2022-09-22 23:14:39
tags:
- Python
- 工具
- TIL
---
网上冲浪看到了一个简单好用的英语单词拼写检查工具 [codespell](https://github.com/codespell-project/codespell)，测试发现真的好用，一键安装&一键开箱使用，没有比这更美好的体验了，下面展开说下流程。
<!--more-->

### 1. 安装
codespell 是用 Python 写的工具，因此直接使用pip安装即可:
```bash
pip install codespell
```
输出应该类似如下：
```bash
Collecting codespell
  Downloading codespell-2.2.1-py3-none-any.whl (202 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 202.1/202.1 kB 165.1 kB/s eta 0:00:00
Installing collected packages: codespell
Successfully installed codespell-2.2.1
```
很简单。

### 2. 使用
进一个包含英文文本的目录，比如你的源码根目录，或者文档目录，然后执行`codespell`, 就会检查当前目录下所有的文本，给出可能的拼写错误。

例如我clone一个我的GitHub 仓库，进去执行`codespell`:
```bash
cd /tmp
git clone https://github.com/vra/easybox
cd easybox
codespell
```
输出结果如下:
```bash
./README.md:10: termial ==> terminal
./README.md:53: termial ==> terminal
./easybox/main.py:41: Mimimal ==> Minimal
```

可以看到，markdown文件和Python文件中的一些拼写错误都被找出来了。


除了这么直接使用外，还可以在命令后面增加一些目录和路径的限定，比如`*.md` 只检查当前目录下的`.md`文件，`folder` 只检查文件夹`folder`下的所有文件，等等，都是Linux下的基本操作。

### 3. 原理
这个工具的大致原理是将英文单词容易出错的情况写到代码库的数据中，然后在代码中进行匹配，所以不会出现别的工具那样，对变量命名的误判断，这是一个很好的特性。具体实现细节就需要查看[源码](https://github.com/codespell-project/codespell)了，有空或许可以分析一下，写一个源码解读哈哈。

上面这些内容，对于普通人日常使用基本是够用了，关于codespell更多高级的配置选项，请参考GitHub上的[README](https://github.com/codespell-project/codespell)文件中的说明。


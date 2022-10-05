---
title: git合并另一个分支的某个文件到当前分支
date: 2022-08-14 11:14:26
tags:
- Git
- Linux
- Python
---
## 概述
使用Git时，有时候不同分支的文件是不同步的，因此如果想要把别的分支的文件改动应用到当前分支，应该怎么操作呢？如果两边都有更新，该如何选择合并呢？这篇小文会对不同情形下的合并进行一个简单的介绍。
<!--more-->

## 引入
假设我们当前在分支`branch1`, 需要将分支`branch2`上的`a.py`合并到当前分支。  
根据[之前写的这篇文章](https://vra.github.io/2021/09/25/git-copy-from-another-branch)，我们可以这么操作
```bash
git checkout branch2 -- a.py
```
## 两边都存在文件
现在换一个情况，假设分支`branch1`和`branch2`都有文件`a.py`，且分支`branch1`上的文件包含在`branch2`的内容里，那么采用上面的命令也还是可以的：
```bash
git checkout branch2 -- a.py
```

另外如果只想合并`branch2`上的文件的一部分更新到`branch1`，可以在`chekcout`后面增加`-p`或者`--patch`选项，交互式地选择要合并过来的代码块:
```bash
git checkout -p branch2 -- a.py
```
交互式地操作命令同`git add -p`，可以参考[这里的文章](https://vra.github.io/2022/06/17/git-add-part-of-a-file/)。


更复杂的情况是，分支`branch1`也有同名文件，且也有更新，如果直接使用`git checkout`的话，分支`branch2`上的文件会替代本地的文件，且没有任何提示（毕竟cheeckout的含义就是切换到某个分支）。因此为了保持本地的更新，需要增加`-p`选项。

这时候，会出现一种情况，本地的更新和远程的更新被放到一个块(hunk)里面，只能保留其中一个，此时就需要更精细的操作，在交互式环境中采用`e`命令来手动对hunk进行更新，去掉或增加代码的`+`或者`-`，具体可以参考[这个回答](https://stackoverflow.com/a/6290646)

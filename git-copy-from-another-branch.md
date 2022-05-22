---
title: git 从别的分支复制文件或目录
date: 2021-09-25 10:46:43
tags:
 - Git
 - Linux
 - 总结
---
有时候我们需要从别的分支复制文件或者目录，这里总结一些简单的命令供查看。
<!--more-->

假设我们的当前分支为`branch1`, 想要复制文件或者目录的分支为`branch2`, 两个分支下文件结构是不同的，具体如下：
branch1: 
```bash
├── README.md
├── cpp
│   ├── include
│   │   └── test.hpp
│   └── src
│       └── test.cpp
└── python
    └── setup.py
```
branch2:
```bash
├── README.md
└── java
    └── test.java
    └── main.java
```

假设我们当前在`branch1`, 目录为仓库根目录，想要复制`branch2` 的 java/test.java` 到当前目录，执行下面的语句:
```bash
git checkout branch2 -- java/test.java
```
**⚠️注意：这里还是会创建一个`java`目录，而不是把`test.java`放到根目录下。**

如果当前进入了`cpp` 子目录，后面的路径也需要改成相对路径:
```bash
git checkout branch2 -- ../java/test.java
```
如果想要复制整个目录，也是一样的:
```bash
git checkout branch2 -- java
```
此外还可以利用提交的hash值来复制文件，这样就会复制当次提交时候的文件内容:
```bash
git checkout 941b6dd java/test.java
```
参考：
1. <https://www.tutsway.com/how-to-copy-file-or-folder-from-one-branch-to-another-in-git.php>

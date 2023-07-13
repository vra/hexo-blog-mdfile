---
title: git diff 的一个妙用
date: 2023-06-30 22:41:49
tags:
 - Git
---
### 1. git diff 常规用法
git diff 可以用来比较在git仓库中的两次提交或两个文件的diff，常见用法如下：

```bash
# 显示当前代码与最新commit的代码之间的差别
git diff

# 显示暂存（也就是已经git add 但还没有git commit）的代码提交
git diff --staged

# 显示当前代码与<commit-id>时代码的区别
git diff <commit-id> 

# 显示暂存代码与<commit-id>时代码的区别
git diff --staged <commit-id> 

# 显示两次commit-id之间的代码区别
git diff <commit-id1> <commit-id2>  

# 显示当前分支与 branch1 分支上的代码区别
git diff <branch1>

# 显示两个分支上的代码之间的区别
git diff <branch1> <branch2>
```

所有上述命令后面都可以加一个目录或文件路径来只显示这个目录或文件中的区别：
```bash
git diff /path/to/folder

git diff /path/to/file.py

# 也可用git的参数终止符号--，避免文件名和参数重名时将文件名解析为参数
git diff --  /path/to/file.py
```
<!--more-->

### 2. git diff 妙用
git diff 有一个选项`--no-index` ，可以用来不在git仓库中的两个文件或目录。
`--no-index`的git帮助文档中说明如下：
```plain
git diff [<options>] --no-index [--] <path> <path>
This form is to compare the given two paths on the filesystem. You can omit the --no-index option when running the command in a working tree controlled by Git and at least one of the paths points outside the working tree, or when running the command outside a working tree controlled by Git. This form implies --exit-code.
```
说明它可以用来比较两个给定的路径。

那为什么要用`git diff` 来比较非git仓库里面的两个路径呢，直接用Linux和Mac上自带的`diff` 命令不好吗？

`git diff` 相比`diff` 的优势是它能生成以`+` 和`-` 开头的diff结果，红色表示删去，绿色表示添加，因此能很直观地看出增加和删除了哪些地方，而diff给出来的是黑色的代码差别，展示很不直观。

另外`git diff`的结果可以写入文件，粘贴到Markdown文件中，大部分 Markdown 渲染器都能够识别diff块，比较好地渲染出diff结果。


实际操作中，需要在一个git仓库目录中来执行`git diff --no-index`,例如比较两个文件:
```bash
git diff --no-index ~/a.py ~/b.py
```
比较两个目录:
```bash 
git diff --no-index ~/folder-a ~/folder-b
```

### One More Thing
其实我之前写过一个比较两个目录的Python工具[dompare](https://github.com/vra/dompare)(名字含义是directory compare)，通过执行一条命令得到得到两个目录中文件的diff，并且保存到HTML网页中打开浏览器进行展示。感兴趣的小伙伴可以玩一玩。

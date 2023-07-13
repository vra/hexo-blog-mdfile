---
title: 原来 git stash 应该这么用
date: 2023-01-15 13:19:57
tags:
- Linux
- Git
- 总结
---
## 概述
前段时间突然发现，我之前对`git stash`的使用都是错误的。

具体说来，我是这么使用的：在远端有新的提交，需要`git pull`来拉取合并时，发现本地有一些未提交的修改，功能也没实现，不适合做一次commit。这时候我执行`git stash`隐藏本地的修改，然后执行`git pull`来拉取远端的更新，在最新代码基础上**重新实现**stash的那些代码中的功能。

这里的问题是，重新实现stash代码中的那一步，其实完全可以用`git stash pop`来替代，执行这个命令会在最新代码基础上作用stash的代码，不用再重新实现一遍了（不过这时可能会有代码冲突需要解决）。所以我之前是把`git stash`当`git checkout -- .`来用了，也就是抛弃了本地的代码更新，显然是有问题的。

正确流程基本上是这样：
```bash
git stash # 或者 git stash push，效果一样
git pull # 可能有冲突需要手动合并
git stash pop # 可能有冲突需要手动合并
```
下面记录一下 git stash 提供的功能和一些参数。
<!--more-->
## git stash 具体用法

`git stash`创建一个新的stash，效果与`git stash push` 一样，效果如下:
```bash
$ git stash
Saved working directory and index state WIP on master: c6771a5 doc: fix error during pre-commiting
```
增加`-u`选项可以将未track的文件也隐藏起来。

你可以创建多个stash，最早的stash表示为`stash@{0}`，然后是`stash@{1}`，依次递加。

`git stash list` 会列出所有的stash：
```bash
$ git stash list
stash@{0}: WIP on master: c6771a5 doc: fix error during pre-commiting
stash@{1}: WIP on master: c6771a5 doc: fix error during pre-commiting
```

`git stash show`可以查看最新stash中的修改，加上编号可以查看之前版本的修改。
```bash
$ git stash show stash@{0}
version.txt | 1 +
 1 file changed, 1 insertion(+)
```
`git stash apply` 可以应用最新的stash到当前的代码中，同样的，如果加上编号则可以应用之前版本的修改到当前代码。apply执行后记得调用`git stash drop` 来去除以及应用的stash。
`git stash pop`效果等于`git stash apply` + `git stash drop`。

`git stash branch`会基于老的提交代码创建一个分支，同时把最新的修改也作用过去，这样对于新的提交和老提交代码变化很大的场景比较好，避免在新的提交上apply stash时由于冲突太多造成的合并问题。

`git stash clean` 会清空所有的stash，**且没有任何提示**，这意味着你所有隐藏的代码都会被删除，执行此命令前请三思！

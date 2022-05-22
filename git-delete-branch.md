---
title: git 删除远程分支
date: 2022-04-10 17:55:35
tags:
- Git
- Linux
---
Git可以方便地删除本地的某个分支。具体操作是：
1. 切换到别的分支
2. 执行`git branch -d <name-of-branch-to-delete>`

比如我想删除当前的`dev-tmp`分支:
```bash
git checkout master
git branch -d dev-tmp
```

上面的命令只删除了本地的分支，如果要删除远端的分支，该怎么操作呢？答案是用带有`--delete`选项的`git push`命令，例如：
```bash
git push origin --delete dev-tmp
```
可以删除远端的`dev-tmp`分支。

---
title: git 回滚代码并保留提交历史
date: 2023-05-16 15:55:28
tags:
- Git
---
在使用git时，有时候需要回退最新代码到之前的某次提交或某个tag，将中间的所有代码提交去掉。同时保持中间的提交记录。实际应用时发现这个动作没有比较好的实现方式。

例如，如果使用`git revert commit-id`, 那么只会会退`commit-id` 对应的那次提交，之后的提交不受影响，仍然存在，不是我们想要的效果。

如果使用`git reset`, 那操作就比较麻烦，需要使用`--hard` 和`--force` 等比较危险的命令，具体如下：
```bash
git reset --hard commit-id
git push --force
```
这样做除了使用比较危险的命令选项外，还有个问题是没法保留中间的提交历史，这不是我们想要的。

搜索发现，利用git diff和git apply可以来比较清晰的完成这个需求，整体的思路是：
1. 得到当前最新提交到回退提交之间的代码diff，将diff保存为文件
2. 利用`git apply` 将diff作用到代码上，回到之前的代码状态
3. 提交代码

具体来说，假设当前最新提交就在分支`current-branch`上，回退提交为`prev-commit`,这个回退提交可以是一次commit id，也可以是一个tag，也可以是一个分支名。执行命令如下：
```bash
git checkout prev-commit
git diff current-branch > ~/diff.patch
git checkout current-branch
cat ~/diff.patch | git apply
git commit -am "roll back to prev-commit"
git push
```

这样就能既回退代码，又保留提交历史。

### 参考
+ <https://stackoverflow.com/a/33890073>

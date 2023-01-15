---
title: git如何添加文件到最新的提交
date: 2022-11-12 17:54:22
tags:
- Git
- TIL
---

有时候，在git commit后，我们会发现一些文件忘了提交了，或者需要修改，而且这些提交和修改是与上一次commit的主题一致的，这时候再执行一遍相同的git commit就会让提交记录显得比较冗余，有没有办法将修改后的文件加到最后一次的提交记录里面呢？搜索后发现[这里](https://stackoverflow.com/a/40503483)给了一个解决办法，git add文件后调用`git commit --amend -no-edit`即可：
```bash
git add <file_path>
git commit --amend --no-edit
```
**注意：如果之前的代码已经提交的话，需要执行`git push --force`来推送代码以替代之前的提交记录。**

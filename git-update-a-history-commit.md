---
title: git 更新历史提交
date: 2022-12-16 09:01:45
tags:
- Linux
- Git
- 总结
---
## 概述
有时候我们在git commit后才发现，之前的一些提交有些问题，比如有些代码忘提交了或者有一些typo需要修改。如果要修改的地方是需要添加到最后一次提交上的，那么可以参考我的[这篇博文](https://vra.github.io/2022/11/12/git-add-file-to-last-commit/)修改，如果是在非最后一次提交上的，那么就需要用`git rebase`来操作。这里简单记录一下操作的过程。

**TL;DR**
操作命令简要来说是这样:
```bash
# 使用git log 查看历史提交，得到需要修改的那次提交的commit id
git log
# 执行rebase命令，注意<commit-id>后面有一个^，表示修改在此次提交前
git rebase -i '<commmit-hash>^' # 如果是修改第一次提交，使用 git rebase -i --root
# 修改代码
vim changed-file
# git add 添加更新后的文件
git add changed-file
# git commit 提交，注意需要使用后面三个选项，并且不需要加commit信息，因为会采用之前的commit信息
git commit --all --amend --no-edit
# 使用--continue来完成 git rebase
git rebase --continue
```
后面会使用一个具体的（假）例子来演示这个过程。
<!--more-->

## 例子
假设我们创建了一个代码仓库`my_project`，先后创建并提交了`README.md`和`main.py`文件，但发现第一次的提交里面有一个typo，例如比`math`打成了`meth`，现在想要修改第一次提交。

首先构造"案发现场":
```bash
mkdir my_project && cd my_project
git init
echo "This is my meth library" >> README.md
git add README.md
git commit -m "doc: add readme"
echo "import numpy as np" >> main.py
git add main.py
git commit -m "feat: create main.py"
```
注意上面的typo `meth`。

我们发现了上述问题，但不想新建一个提交来修复，因为确实不算是新功能，那么就用`git rebase`来完成吧。

git rebase 是用来修改git commit的命令，提供了非常多的功能。这里我们用`git rebase -i`来交互式地修改某次commit。

首先用 `git log`查看commit ID：
```bash
$ git log

* 9bec788 - (HEAD -> main) add sigmoid (31 minutes ago) <xyz>
* ea833e9 - doc: add doc (31 minutes ago) <xyz>
```
假如要修改第二次提交，那我们可以用`git rebase -i '9bec788^`，但我们要修改的是第一次提交，没有之前的状态，所以要用下面的命令:
```bash
$ git rebase -i --root
Successfully rebased and updated refs/heads/main.
```

出来的交互式界面:
```bash
pick ea833e9 doc: add doc
pick 9bec788 add sigmoid

# Rebase 9bec788 onto 927493a (2 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
#         create a merge commit using the original merge commit's
#         message (or the oneline, if no original merge commit was
#         specified); use -c <commit> to reword the commit message
# u, update-ref <ref> = track a placeholder for the <ref> to be updated
#                       to this position in the new commits. The <ref> is
#                       updated at the end of the rebase
#
# These lines can be re-ordered; they are executed from top to bottom.
```

底下注释中给出了rebase支持的一些命令和对应的缩写，我们将需要修改的提交前面的命令修改为`edit`，然后保存退出:
```bash
edit ea833e9 doc: add doc
pick 9bec788 add sigmoid

# Rebase 9bec788 onto e3f4cea (2 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
#         create a merge commit using the original merge commit's
#         message (or the oneline, if no original merge commit was
#         specified); use -c <commit> to reword the commit message
# u, update-ref <ref> = track a placeholder for the <ref> to be updated
#                       to this position in the new commits. The <ref> is
#                       updated at the end of the rebase
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
```
保存后输出如下：
```bash
Stopped at ea833e9...  doc: add doc
You can amend the commit now, with

  git commit --amend

Once you are satisfied with your changes, run

  git rebase --continue
```
用`git status` 查看代码状态:
```bash
interactive rebase in progress; onto e3f4cea
Last command done (1 command done):
   edit ea833e9 doc: add doc
Next command to do (1 remaining command):
   pick 9bec788 add sigmoid
  (use "git rebase --edit-todo" to view and edit)
You are currently editing a commit while rebasing branch 'main' on 'e3f4cea'.
  (use "git commit --amend" to amend the current commit)
  (use "git rebase --continue" once you are satisfied with your changes)

nothing to commit, working tree clean
```
git 的提示信息还是很丰富的，按照提示来操作代码，将`meth` 修改为`math`，再`git add`, `git commit --all --amend --no-edit`和 `git rebase --continue` 来结束rebase:
```bash
$ git add README.md

$ git commit --all --amend --no-edit
[detached HEAD 3b83a85] doc: add doc
 Date: Sat Dec 17 18:00:12 2022 +0800
 1 file changed, 3 insertions(+)
 create mode 100644 README.md

$ git rebase --continue
Successfully rebased and updated refs/heads/main.
```
然后用`git log`查看命令，可以看到修改的那次提交和后续提交的编号都已经更新了，意味着这是全新的提交，跟之前的提交没有关系了。

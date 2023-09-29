---
title: git clean 教程
date: 2023-07-30 00:21:01
tags:
 - Git
 - 总结
---
### 1. 引入
git clean 是用来删除 git 仓库中没有被跟踪的文件的命令，在想要快速清理 git 仓库（比如，删除仓库中所有没有跟踪的文件，清除编译生成的临时文件）时很有用。是相比别的git子命令， git clean的配置选项比较少，使用起来简单一些，这里写一个简要教程。
友情提示：git clean真的会删除文件，而且没法用git命令来恢复（因为没有被 git 跟踪），所以使用git clean前务必慎重，建议每次删除文件之前先加`--dry-run` 选项来验证会删除哪些文件，确保没有误删。
<!--more-->


### 2. git clean 选项的含义
先创建一个简单的git 仓库环境来比较清晰地展示各个选项的效果:
```bash
mkdir /tmp/git_clean_demo
cd /tmp/git_clean_demo
git init
touch a.py b.py
git add a.py
mkdir -p folder0/folder00
mkdir -p folder0/folder01
touch folder0/folder0.py
touch folder0/folder00/folder00.py
touch folder0/folder01/folder01.py
git add folder0/folder0.py
git add folder0/folder00/
touch folder0/folder00/folder00_v2.py
echo "*.pyc" >> .gitignore
touch a.pyc
git add .gitignore
```

用`git status` 查看一下文件跟踪状态：
```bash
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   a.py
        new file:   folder0/folder0.py
        new file:   folder0/folder00/folder00.py

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        b.py
        folder0/folder00/folder00_v2.py
        folder0/folder01/
```

在 Git 的800多个配置选项中，只有一项是关于`git clean` 命令的：`clean.requireForce`。这个选项的意思是，使用`git clean` 时，必须加`-f`或者`--force` 参数才能删除文件，否则并不会删除文件，执行时会提示下面信息：
```bash
$ git clean
fatal: clean.requireForce defaults to true and neither -i, -n, nor -f given; refusing to clean
```
这是一个很好的保护文件不被轻易删除的选项，建议不要修改默认值。

所以 `-f/--force`的选项的含义就是强制删除，实际删除文件时必带此选项。

`-n/--dry-run`表示不实际删除任何东西，只是空跑一下，用来看哪些文件会被删除掉。对于这种破坏性的命令，增加`--dry-run`选项真的是一个非常好的设定。


另一个很重要的选项是`-d`，表示进入**未跟踪**的目录来递归删除文件。注意对已经跟踪的目录，不加`-d` 命令也会清理其中的未跟踪文件，一定注意！
比如刚才创建的git仓库，不加`-d` 选项删除时结果如下:
```bash
$ git clean -f --dry-run
Would remove b.py
Would remove folder0/folder00/folder00_v2.py
```

加了 `-d`选项：
```bash
$ git clean -f -d --dry-run 
Would remove b.py
Would remove folder0/folder00/folder00_v2.py
Would remove folder0/folder01/
```
可以看到不管加不加`-d`，已经跟踪的目录下的未跟踪文件都会被删除；而只有加了`-d`，未跟踪的目录和下面的文件才会被删除。

`-q/--quiet`表示静默操作，除了错误，别的信息不显示，实际效果：
```bash
$ git clean -f -d -q --dry-run
```
可以看到没有任何输出。

`-i/--interactive` 表示交互式地删除文件，用于对文件删除进行精细操作。进入交互式界面后，又可以分按模式删除、按数字删除、每次删除前询问几种方式，具体看下面的交互式会话：
```bash
$ git clean -f -d -i --dry-run
Would remove the following items:
  b.py                             folder0/folder00/folder00_v2.py  folder0/folder01/
*** Commands ***
    1: clean                2: filter by pattern    3: select by numbers    4: ask each             5: quit                 6: help
What now> h
clean               - start cleaning
filter by pattern   - exclude items from deletion
select by numbers   - select items to be deleted by numbers
ask each            - confirm each deletion (like "rm -i")
quit                - stop cleaning
help                - this screen
?                   - help for prompt selection
Would remove the following items:
  b.py                             folder0/folder00/folder00_v2.py  folder0/folder01/
*** Commands ***
    1: clean                2: filter by pattern    3: select by numbers    4: ask each             5: quit                 6: help
What now> c
Would remove b.py
Would remove folder0/folder00/folder00_v2.py
Would remove folder0/folder01/
```

按规则忽略文件，也就是匹配到规则的图片不进行删除：
```bash
$ git clean -f -d -i --dry-run
Would remove the following items:
  b.py                             folder0/folder00/folder00_v2.py  folder0/folder01/
*** Commands ***
    1: clean                2: filter by pattern    3: select by numbers    4: ask each             5: quit                 6: help
What now> f
  b.py                             folder0/folder00/folder00_v2.py  folder0/folder01/
Input ignore patterns>> *folder*
  b.py
Input ignore patterns>>
Would remove the following item:
  b.py
*** Commands ***
    1: clean                2: filter by pattern    3: select by numbers    4: ask each             5: quit                 6: help
What now> c
Would remove b.py
```

按数字删除：
```bash
$ git clean -f -d -i --dry-run
Would remove the following items:
  b.py                             folder0/folder00/folder00_v2.py  folder0/folder01/
*** Commands ***
    1: clean                2: filter by pattern    3: select by numbers    4: ask each             5: quit                 6: help
What now> s
    1: b.py                               2: folder0/folder00/folder00_v2.py    3: folder0/folder01/
Select items to delete>> 1
  * 1: b.py                               2: folder0/folder00/folder00_v2.py    3: folder0/folder01/
Select items to delete>> 2
  * 1: b.py                             * 2: folder0/folder00/folder00_v2.py    3: folder0/folder01/
Select items to delete>> 3
  * 1: b.py                             * 2: folder0/folder00/folder00_v2.py  * 3: folder0/folder01/
Select items to delete>> 4
Huh (4)?
  * 1: b.py                             * 2: folder0/folder00/folder00_v2.py  * 3: folder0/folder01/
Select items to delete>> 5
Huh (5)?
  * 1: b.py                             * 2: folder0/folder00/folder00_v2.py  * 3: folder0/folder01/
Select items to delete>>
Would remove the following items:
  b.py                             folder0/folder00/folder00_v2.py  folder0/folder01/
*** Commands ***
    1: clean                2: filter by pattern    3: select by numbers    4: ask each             5: quit                 6: help
What now> c
Would remove b.py
Would remove folder0/folder00/folder00_v2.py
Would remove folder0/folder01/
```
注意看输入大于未跟踪文件数目的数字时的`Huh`，有点喜感。

`-e/--exclude`表示删除时排除满足后面模式的文件，比如`-e "*/"` 表示排除所有文件夹，`-e "*_v2.py"`表示排除所有以`_v2.py`结尾的文件：
```bash
$ git clean -f -d -e "*/" --dry-run
Would remove b.py

$ it clean -f -d -e "*_v2.py" --dry-run
Would remove b.py
Would remove folder0/folder01
```

`-x`  表示不使用.gitignore中的规则。如果不加这个选项，默认会跳过.gitignore 规则中的文件，启用这个选项后会将`.gitignore` 中的文件也删除，比如创建示例仓库时我们忽略了`*.pyc`，前面的结果中都没跳过了这一类文件，加了`-x`选项后输出如下:
```bash
$ git clean -f -d -x --dry-run
Would remove a.pyc
Would remove b.py
Would remove folder0/folder00/folder00_v2.py
Would remove folder0/folder01/
```
`a.pyc` 也被删除掉了。

`-X`选项（大写的X）与`-x` 相反，只删除满足`.gitignore` 中规则的文件：
```bash
$ git clean -f -d -X --dry-run
Would remove a.pyc
```

### 3. 总结
实际实践中，我对`git clean` 用的还不多(严格来说正经使用只用过一次)，本文中如果错误，欢迎批评指正。

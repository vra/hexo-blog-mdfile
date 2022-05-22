---
title: Git禁止大文件提交到仓库中
date: 2019-03-10 12:23:54
tags:
 - Git
 - Linux
 - 总结
---
## 概述
Git提交的时候，有的时候很容易将目录下的非源代码的文件（如二进制文件、模型等）提交到Git仓库里，给后续的使用造成麻烦。那么有没有一种方法来限制提交到Git的文件的大小呢，答案是Yes，下面我来大概介绍下吧。
<!--more-->
原理是利用Git的[钩子](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)来在`commit`之前执行一个脚本，在这个脚本里对提交的文件大小进行检查。

具体操作是：修改仓库下的`.git/hooks/pre-commit`为如下内容（如果没有这个文件请新建）：
```bash
#!/bin/sh
hard_limit=$(git config hooks.filesizehardlimit)
soft_limit=$(git config hooks.filesizesoftlimit)
: ${hard_limit:=10000000} # 10M
: ${soft_limit:=1000000} # 1M

list_new_or_modified_files()
{
    git diff --staged --name-status|sed -e '/^D/ d; /^D/! s/.\s\+//'
}

unmunge()
{
    local result="${1#\"}"
    result="${result%\"}"
    env echo -e "$result"
}

check_file_size()
{
    n=0 
    while read -r munged_filename
    do
        f="$(unmunge "$munged_filename")"
        h=$(git ls-files -s "$f"|cut -d' ' -f 2)
        s=$(git cat-file -s "$h")
        if [ "$s" -gt $hard_limit ]
        then
            env echo -E 1>&2 "ERROR: hard size limit ($hard_limit) exceeded: $munged_filename ($s)"
            n=$((n+1))
        elif [ "$s" -gt $soft_limit ]
        then
            env echo -E 1>&2 "WARNING: soft size limit ($soft_limit) exceeded: $munged_filename ($s)"
        fi
    done

    [ $n -eq 0 ] 
}

list_new_or_modified_files | check_file_size
```
这里设置了`soft_limit`和`hard_limit`，默认的大小分别是1M和10M，当提交的某个文件超过1M时，会显示警告；当超过10M时，会显示错误，导致commit失败。

此外，可以通过`git config`命令来设置`soft_limit`和`hard_limit`的值：
```bash
git config hooks.filesizehardlimit 20000000
git config hooks.filesizesoftlimit 2000000
```
请根据自己的使用情况酌情修改具体的数值。

需要注意的是，`.git`目录下的文件Git是没有跟踪的，因此在别的电脑或目录下`git clone`仓库后，`pre-commit`文件并不会被自动clone进来，需要手动添加。

我在[GitHub Gist](https://gist.github.com/vra/ae91ade1e15ce31b42f6366a91d1ac17)上提交了这个文件，有需要的小伙伴可以直接下载使用。

## 参考
 1. <https://stackoverflow.com/questions/39576257/how-to-limit-file-size-on-commit>
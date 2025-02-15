---
title: 国内加速 GitHub 代码克隆的一种方案
date: 2024-09-14 08:47:18
tags:
  - Git
  - GitHub
---

国内下载 GitHub 上代码一直是一件让人很头疼的事情，相信大家都深有体会。

最近偶然发现一个比较好用的解决方案，是采用<http://gitclone.com>的加速，这里记录一下。

具体来说，在仓库url中增加`gitclone.com`的前缀，别的地方不变，即`https://github.com/`修改为`https://gitclone.com/github.com/`，例如原始的clone命令是:

```bash
git clone https://github.com/huggingface/transformers
```
替换成下面的命令即可：

```bash
git clone https://gitclone.com/github.com/huggingface/transformers
```

实测基本上能做到1M/s的下载速度。

这种加速目前只支持git clone 和git pull 命令，所以适用于拉取别人代码进行本地查看的应用场景。

另外发现这种加速方式下载的仓库，有一些只有最新的一次提交，有一些则包含完整提交，原因未知。

此外，请确认克隆的代码是否与GitHub上一致，我们无法保证拉取的代码是否被修改过。

---
title: Linux小知识:apt install什么时候会出现 Do you want to continue?的提示
date: 2022-07-23 11:17:27
tags:
- Linux
- Ubuntu
---

## 1. 一个故事
在大约十年以前，大二或大三的夏天，[子浩](https://wzhd.org/) 在科大西区活动中心的自习室内，给我帮忙安装Linux系统。当时他问了我一个问题，执行`apt-get install` 安装软件时，什么时候会弹出`Do you want to continue?Y/[n]`的提示呢？我当时说应该是包大小超过某个限制大小时会有这个提示吧，但我们随即验证了这个假设并不成立，安装一个很小的包也会有这个提示。当时没有得到明确的结论。

这个问题时不时在脑海中想起，每次想查一下弄个清楚，但始终没来得及调查。这样十年过去了，近日终于又想起来，通过谷歌搜索，发现StackOverflow上已经有人问过同样的问题，而且也有人进行了回答。至此这个小疑问算是解决了，我也不用每次念念不忘了。

具体原因是什么，请见下节。
<!--more-->


## 2. 谜底揭晓
根据[这里](https://superuser.com/questions/287348/why-does-apt-get-sometimes-asks-for-confirmation/287357#287357)的回答，`Do you want to continue` 的提示会在下面几种情况下出现:
1. 当除了你要安装包外，有额外的依赖包需要被安装时，比如你执行的是`sudo apt install aaa`, 包 `aaa` 依赖 `bbb`，因此`bbb` 也需要被安装，这时候就会有提示
2. 当现有包的版本要改变，比如机器上已经安装了`bbb-1.0.0`，而安装`aaa`需要安装`bbb-2.0.0`，这时候就会有提示
3. 当基础包要被移除的时候，基础包大体意思是系统运行所需的最小软件包集，删除了就会导致系统运行出问题，详细的概念可以参考[这里](https://www.debian.org/doc/debian-policy/ch-binary.html#essential-packages)

常见的情况应该是第一种和第二种，第三种情况应该是只有`apt remove`才会出现。

## 3. 参考
1. <https://superuser.com/questions/287348/why-does-apt-get-sometimes-asks-for-confirmation/287357#287357>
2. <https://unix.stackexchange.com/questions/70651/when-does-apt-get-install-ask-me-to-confirm-whether-i-want-to-continue-or-not>




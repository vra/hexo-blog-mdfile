---
title: 加速国内访问 Arxiv 论文的一些方法
date: 2020-03-01 09:32:12
tags:
 - 总结
 - Linux
---
arxiv 的 PDF 下载速度很慢，下面是一些加速方法。
## 命令行直接下载
我们知道可以用`wget`命令下载一些网络文件， 不过arxiv 上的论文使用`wget`下载时需要加参数`--user-agent=Lynx`，速度才能较快，下面是使用的例子：
```bash
wget --user-agent=Lynx https://arxiv.org/pdf/1911.05722.pdf
```
上述命令需要在Linux或者WSL的命令行中执行。

## 修改网址
一种方法是将`https://arxiv.org`改成 `http://xxx.itp.ac.cn`，后面内容不变，速度飞快。
还有一种方式是将`https://arxiv.org`改成`http://cn.arxiv.org`，后面网址内容不变，不过这个方法有时候并不work，因此推荐上一种方法。

更多方法可以参考知乎上的[这个问题](https://www.zhihu.com/question/58912862)。
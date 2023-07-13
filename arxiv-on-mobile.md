---
title: 手机上看arxiv上论文的方法
date: 2023-01-27 22:44:41
tags:
- Latex
- HTML
- Web
- 总结
---
有时候想要在手机上访问Arxiv上的论文，打开arxiv.com，发现体验比较差，没有响应式设计，需要不断移动页面才能读完一行文字，影响阅读。偶然发现了[arxiv-vanity](https://www.arxiv-vanity.com/)这个网站，发现能很好的满足手机上看arxiv论文的需求，收藏了。

<!--more-->
首先看下arxiv-vanity网站的介绍:
> arXiv Vanity renders academic papers from arXiv as responsive web pages so you don’t have to squint at a PDF.

翻译成中文就是:
> arXiv Vanity 将 arXiv 的学术论文呈现为响应式网页，因此您不必眯着眼睛看 PDF。

exactly what I need!

那么该如何使用呢？

在[arxiv-vanity](https://www.arxiv-vanity.com/)首页的搜索框中输入arxiv论文的摘要页面，如`https://arxiv.org/abs/1605.07683`，按右边的按钮，就能将论文转换为HTML文件，并且在不同的设备下自适应地调整大小。

另外也可以通过`https://www.arxiv-vanity.com/papers/<paper_id>`的方式访问转换后的HTML页面，比如`https://www.arxiv-vanity.com/papers/1605.07683/`。

大概原理是使用[LaTeXML](https://dlmf.nist.gov/LaTeXML/)将Latex原文件转换为HTML，再进行显示。具体实现参见[源码](https://github.com/arxiv-vanity/engrafo)。

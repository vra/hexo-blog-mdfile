---
title: 用 Material for MkDocs 来生成专业的技术文档
date: 2023-05-17 19:31:37
tags:
- Docs
- Markdown
---

## 1. 概述
对于程序员来说，写技术文档是一项必备的技能。由于GitHub和Markdown格式的普及，很多时候我们可以用markdown来简便地写出技术文档，并且 通过GitHub Pages等工具快速地进行技术文档的部署。

虽然GItHub Pages默认支持静态文档框架[Jekyll](https://jekyllrb.com/)，也包含一些简单的[主题](https://pages.github.com/themes/)，但对于文档和教程比较多的项目来说，使用GitHub Pages的默认部署工具还不够用，主要体现在下面几个方面：
+ Markdown本身支持的语法比较简单，一些复杂的像Warning等提示没法直接用Pages的默认主题来实现
+ Pages 默认显示的是单页文档，没有侧边栏、导航栏等工具
+ Pages 默认主题无法搜索文档内容
+ Pages 不支持选择`Linux`或`Windows` 后显示不同执行语句的功能
+ ...

[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) 是 [MkDocs](https://www.mkdocs.org/)的一个主题配置，同时也是一个功能齐全的静态网站生成工具，能够解决上面提到的GitHub Pages的问题。

Material for MkDocs 使用广泛，下面是一些大公司和知名开源项目的使用例子：
+ [AWS Copilot CLI ](https://aws.github.io/copilot-cli/)
+ [Google Accompanist](https://google.github.io/accompanist/)
+ [MicroSoft Code With Engineering Playbook](https://microsoft.github.io/code-with-engineering-playbook/)
+ [Mozilla Foundation Engineering Handbook](https://mozillafoundation.github.io/engineering-handbook/)
+ [Netflix Titus](https://netflix.github.io/titus/)
+ [CentOS Infra docs](https://docs.infra.centos.org/)
+ [electron-builder](https://www.electron.build/)
+ [Kubernetes](https://kops.sigs.k8s.io/)


虽然我还没有比较复杂的开源项目需要用mkdocs-material来管理文档，但看到GitHub Pages的一些限制，最近有空还是学了一下这个工具，以备后续项目中使用。这里做一些简单记录，方便以后查找。

需要说明的是，Material for MkDocs 是一个比较复杂的工具，很多配置项这里没有提到，根据需要在官方[Setup](https://squidfunk.github.io/mkdocs-material/setup/)文档中查看使用说明。

另外一种学习配置的方式是直接查看上面提到的开源项目源码根目录下的`mkdocs.yml`文件，复制这个文件过去，就能得到类似的布局效果。

这个教程里面的示例页面：<https://vra.github.io/mkdocs-material-example/>
示例页面的配置文件：<https://github.com/vra/mkdocs-material-example/blob/main/mkdocs.yml>

<!--more-->

## 2.1 安装
可以直接使用 `pip` 来安装：
```bash
pip install mkdocs-material
```

使用下面的命令测试是否安装成功：
```bash
mkdocs -h
```

其他从docker安装、从GitHub安装的方式参考[官方文档](https://squidfunk.github.io/mkdocs-material/getting-started/#with-docker)。

### 2.2 使用
mkdocs-material 的使用命令比较简单，概括来说就是三板斧：
1. `mkdocs new .`： 在当前目录生成`docs`目录和`mkdocs.yml` 配置文件
2. `mkdocs serve`： 在本地运行文档生成服务，可在浏览器中访问`localhost:8000`查看文档的效果
3. `mkdocs build`： 非必需，在`sites` 目录中生成最终的HTML文件

由于命令比较简单，没有什么太多东西，因而核心要做的事情其实是：
+ 写markdown 格式的文档文件
+ 修改配置文件`mkdocs.yml`

在`mkdocs serve` 运行的过程中，更新完 `mkdocs.yml`配置文件后，文档生成效果实时更新。

### 2.3 上传文档到 GitHub Pages
mkdocs-material 一个很棒的特性是可以一键将代码部署到GIthub Pages上，并且通过GitHub Actions配置，Push 代码时自动更新文档。
假如你的GitHub 仓库地址是`https://github.com/user/repo`，那完成配置后你就可以在`https://user.github.io/repo` 网址查看你的mkdocs-material 文档了。

具体来说，假设你已经创建了一个Git 仓库，需要做下面的事情：
1. 将`mkdocs.yml` 和`docs` 目录提交到Git仓库
2. 增加GitHub Action 配置文件`.github/workflows/ci.yml`，写入下面的内容并提交到GitHub:
```yml
name: ci 
on:
  push:
    branches:
      - master 
      - main
permissions:
  contents: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: 3.x
      - run: echo "cache_id=$(date --utc '+%V')" >> $GITHUB_ENV 
      - uses: actions/cache@v3
        with:
          key: mkdocs-material-${{ env.cache_id }}
          path: .cache
          restore-keys: |
            mkdocs-material-
      - run: pip install mkdocs-material 
      - run: mkdocs gh-deploy --force

```
3. 在GitHub仓库的`Settings` -> `Pages` -> `Build and deployment` 部分，Source 选项选择"Deploy from a branch", Branch 选择`gh-pages`, folder选择`/(root)` 
经过这个配置后，每次向`master` 或`main` 分支push代码，会自动更新`user.github.io/repo`下的文档。

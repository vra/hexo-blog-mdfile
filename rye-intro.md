---
title: Rye:一个实验性质的Python包管理系统
date: 2023-05-17 10:08:32
tags:
- Python
- Rust
---
[Rye](https://mitsuhiko.github.io/rye/) 是[Flask](https://flask.palletsprojects.com/en/2.3.x/)的作者[Armin Ronacher](https://github.com/mitsuhiko)最近推出的一个实验性质的Python包管理系统，目的是解决Python包管理目前面临的工具链碎片化的问题。

大家知道，Python目前的包管理系统很多，包括 poetry, pip, pipenv, pyenv, venv, virtualenv, pdm, hatch 等等，它们都是优秀的工具，提出时都是解决了一定的问题，但没有哪个工具能够做到主流，因此也增加了系统的碎片化程度。

另一方面，conda等工具能提供不同版本的 Python，管理不同的环境，但每个环境的 Python 不是共享的，环境创建一多，环境目录就变得很大，且内部机制很不透明，有时也会遇到冲突没法解决的问题。

另一方面，Python 在Linux/macOS上的安装也面临一些问题，例如用包管理器安装的  Python和用户手动安装的 Python 有的时候会混淆，导致一些混乱，例如在 Fedora 上，用`pip install` 安装包可能会导致系统的包管理命令`dnf` 出错。[PEP 668](https://peps.python.org/pep-0668)尝试对这些问题给出一个解决方案，但也需要不同的系统来支持，目前看还任重道远。

由于Armin也是一个Rust 开发者，而Rust基于标准化的`rustup`和`cargo`两个工具，配合配置文件来进行包管理，目前做的比较好，没有Python面临的碎片化问题。受Rust的启发，作者提出了Rye，并且期望能够启发Python社区提出类似Rust的标准包管理工具。

具体来说，Rye 提出了一些解决这些问题的思路：
+ 提出一个workspace的概念，workspace类似一个项目目录，或者一个git仓库，一个workspace下只有一个Python版本，不同workspace Python版本相互隔离，每个项目采用`pyproject.toml`来进行配置
+ 不使用系统自带的Python，相反地，在每个项目目录的中下载一个standalone的python，解决不同版本的冲突问题
+ 不暴露pip命令，通过`rye add` + `rye sync` 来管理包的依赖，避免包A和包B依赖不同版本的包C而导致的不兼容问题
+ 区分开发环境和正式环境，因为一些包在开发时会用到一些调试工具，但作为第三方库被引入的时候并不需要
+ 支持import本地workspace作为第三方库包

但同时也有一个问题：rye会不会是另一个做不到主流的Python包管理系统，从而进一步增加Python包管理的碎片化呢？作者也有这个考虑，因此写了一个讨论帖 [Should Rye Exist?](https://github.com/mitsuhiko/rye/discussions/6)来讨论这个问题，同时关于Rye的设计初衷，可以参考[这里](https://mitsuhiko.github.io/rye/philosophy/)作者的思考。

个人观点：Rye的出现给Python社区引入了一些新鲜的解决现有问题的思路。使用Rye一段时间后，发现至少使用standalone 的Python版本是一个解决冲突的好的方式。通过几个简单的命令来解决版本管理的问题是比较直观的，提出Rye应该是利大于弊的，也就是有益程度大于碎片化增加的程度。

总之不管是[PEP 668](https://peps.python.org/pep-0668)中标记版本管理是系统的还是Python的，还是[PEP 711](https://peps.python.org/pep-0711/)来单独下发Python解释器二进制文件，还是Rye的出现，都是Python社区意识到Python包管理问题的严重性，进而做出的一些有益尝试。期待在未来，有更标准化的工具，Python的开发也更容易。

下面将对Rye的安装和使用进行简单介绍。
<!--more-->
### 2.1 安装rustup

Rye是基于Rust 开发的，而Rust 有标准的包安装工具`cargo`，Rust编译器和`cargo`都需要用`rustup`来安装，因此安装预编译的Rye包需要先安装`rustup`:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

执行完后，重启Shell，输入`cargo -V`，如果不报错，说明安装成功。

### 2.2 安装Rye
有了cargo后，使用下面的命令安装Rye:
```bash
cargo install --git https://github.com/mitsuhiko/rye rye
```
通过命令行执行`rye -h` 来判断 Rye是否安装成功。

同时可以将`$HOME/.rye/shims` 添加到环境变量`PATH` 中，这样打开Shell后运行`python` 就用的是Rye安装到standalone Python，否则你需要用`rye run python` 来启用Rye的Python解释器。

更新Rye到最新版：
```bash
rye self update
```

删除Rye:
```bash
cargo uninstall rye
```

### 2.3 初始化一个Rye项目
使用`rye init project-name` 来创建一个Rye项目目录

```bash
rye init test_rye
cd test_rye
tree
```

输出如下：
```bash
├── .git
├── .gitignore
├── .python-version
├── README.md
├── pyproject.toml
└── src
    └── test_rye
        └── __init__.py
```

可以看到创建了.git 目录， .gitignore 文件，README.md，配置文件`pyproject.toml` 和一个示例的源码文件`src/test_rye/__init__.py`。

### 2.4 Python 版本管理
为了固定开发环境，我们可以利用`rye pin python-version` 来固定Python的版本，例如`rye pin cpython@3.10.11` 会将Python版本固定为3.10.11。
```bash
# cpython@可以省略
rye pin cpython@3.10.11
rye pin 3.10.11
```
由于默认使用的Python版本是Cpython的，因此在执行rye命令时可以将`Cpython@` 前缀省去。

注意 `rye pin`命令并不立即改变Python的版本，只是修改配置文件`.python-version`，在`rye sync` 执行时才进行实际的修改。

可以多次执行`rye pin` 来调整Python的版本。

然后执行`rye sync` 来同步配置，具体来说，第一次执行这个命令的时候，Rye会下载一个单独的Python解释器，放置到`$HOME/.rye/py`目录下，链接到项目的`.venv` 目录下，因此同一个Python版本在磁盘上只有一份，这与conda是不同的。

更一般地，可以使用`rye toolchain` 来查看、拉取和删除Python版本。

`rye toolchain list` 用来显示所有已经安装的Python版本：
```bash
rye toolcahin list
```
输出：
```bash
cpython@3.11.3 (/Users/yunfeng/.rye/py/cpython@3.11.3/install/bin/python3)
cpython@3.11.1 (/Users/yunfeng/.rye/py/cpython@3.11.1/install/bin/python3)
cpython@3.10.11 (/Users/yunfeng/.rye/py/cpython@3.10.11/install/bin/python3)
cpython@3.10.9 (/Users/yunfeng/.rye/py/cpython@3.10.9/install/bin/python3)
```

`rye toolchain list --include-downloadable` 会列出所有可以下载的Python版本：
```bash
`rye toolchain list --include-downloadable` 
```
输出：
```bash
cpython@3.10.8 (downloadable)
cpython@3.10.7 (downloadable)
cpython@3.10.6 (downloadable)
cpython@3.10.5 (downloadable)
cpython@3.10.4 (downloadable)
cpython@3.10.3 (downloadable)
cpython@3.10.2 (downloadable)
cpython@3.10.0 (downloadable)
...
```
注意已经下载的Python版本不在这个输出中。

`rye toolchain fetch`（简写为`rye fetch`) 可以直接拉取某个Python版本:
```bash
rye toolchain fetch 3.8.16
```

`rye toolchain remove` 可以删除某个Python版本：
```bash
rye toolchain remove 3.8.16
```

### 2.5 添加依赖包
可以通过`rye add package-name` 来安装像numpy等第三方，这个命令支持安装GitHub和本地的包，一些示例的用法如下:
```bash
rye add numpy
# 同时安装几个包
rye add six easydict
# 设置安装包的版本
rye add "Flask>=2.0"
# 只在development环境添加包
rye add --dev black
# 添加github上的包
rye add Flask --git=https://github.com/pallets/flask
# 添加本地目录的包
rye add My-Utility --path ./my-utility
```
同样的，`rye add`并不会实际安装包，只会修改配置文件`pyproject.toml` 中的`dependencies` 项，等执行`rye sync`的时候才真正安装。

### 2.6 Rye工作流
我自己探索的Rye工作流大概是这样：
1. `rye init project-name` 来初始化项目目录
2. `git add` 和`git commit` 来提交初始状态的代码，方便定位后续代码和配置文件的更新
3. `rye pin` 指定Python版本
4. 修改代码，`rye add package-name` 来增加代码依赖的包
5. `rye sync`来安装Python，安装依赖包，更新配置文件
6. `rye run python` 执行代码测试
7. 可选：`rye build` 来生成可发布的wheel文件
8. 可选：`rye publish` 上传包到pypi

需要注意的是，Rye只负责依赖管理，具体的调试代码工作，还需要自己来进行，使用你熟悉的代码测试方式就可以了。

额外补充一下，可以使用`rye shell` 来打开一个新的启用了Rye Python的Shell来进行代码调试。

### 2.7 安装可执行的 global Python工具
某些python包除了包含Python源码外，还包含一些命令行工具，Rye称这些工具为`global tool` ，因为它们不是在某个环境中才能使用，而是全局可使用的。这些工具可以用`rye install package-name`来安装，例如:
```bash
rye install black
```
使用方式为`rye run tool-name`:
```bash
rye run black -h
```

这些包都存放在`$HOME/.rye/shims` 目录下。
如果要删除 global tool，可以使用`rye uninstall`:
```bash
rye uninstall black
```

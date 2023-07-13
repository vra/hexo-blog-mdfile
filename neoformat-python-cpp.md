---
title: NeoVim 代码格式化教程
date: 2023-06-17 09:54:45
tags:
  - Vim
  - NeoVim
  - Python
  - black
  - C++
  - clang-format
---
### 1. 概述
[neoformat](https://github.com/sbdchd/neoformat) 是 (Neo)Vim 的代码格式化插件，支持多种语言的格式化。这篇文章覆盖 Neoformat 对 Python 和 C++ 进行格式化的配置，以及如何在保存代码时自动进行格式化，可以直接应用的配置代码段在文章最后。

<!--more-->

### 2. neoformat安装
采用 [Vim-Plug](https://github.com/junegunn/vim-plug) 进行插件管理，在`~/.config/nvim/init.vim` 中添加下面的插件:
```vimscript
Plug 'sbdchd/neoformat'
```
然后用`:PlugInstall` 命令来安装插件。由于插件源码在 GitHub 上，国内访问时断时续，一次执行可能安装不成功，可以多执行几次这个命令，直到输出窗口显示安装成功。

### 3. neoformat 格式化 Python 代码
#### 3.1 安装格式化工具
neoformat本 身不会安装格式化工具，它只会调用系统已经安装好的格式化工具来进行代码格式化，所以你还需要自己手动在系统上安装格式化工具。

以 Python 格式化为例，我们采用 black 来格式化代码，那么需要先用`pip` 命令来安装black:
```bash
python3 -m pip install black
```
然后需要确保在命令行执行`black --version` 命令能正常输出，neoformat 才能找到black:
```bash
$ black --version
black, 23.3.0 (compiled: yes)
Python (CPython) 3.8.16
```
#### 3.2 格式化配置
安装好以后，我们就可以在`~/.config/nvim/init.vim` 文件中进行 neoformat 配置:
```vimscript
let g:neoformat_python_black = {
            \ 'exe': 'black',
            \ 'args': ['-q', '-'],
            \ 'stdin': 1,
            \ }

let g:neoformat_enabled_python = ['black']
```
这是 VimScript 的语法，`let g:neoformat_python_black` 是创建一个全局变量`neoformat_python_black`, 全局变量的特点是所有打开的窗口和缓冲区都可以访问该变量。

注意这个变量的命名方式，`neoformat_<Language>_<formatter>`，表示针对某个语言的某一个格式化工具，这个格式化工具的名字会被注册，在下面的enable语句中使用到。

全局变量的值的含义如下：
+ `exe` 表示格式化运行需要执行的程序名，就跟我们在命令行访问某个程序一样的机制，需要知道它叫什么才能来执行。
+ `args` 表示程序执行时需要的参数。这里`-q`是black命令的参数项，表示静默执行，不打印输出；`-` 表示从标准输入读取内容来格式化
+ `stdin`: 这个参数表示是否从标准输入来读取内容来格式化。标准输入对应的是文件的内容，除了标准输入外还有缓存区

所有的可配置参数参考 [neoformat 文档](https://github.com/sbdchd/neoformat#config-optional)。这里我们配置这几个参数项就可以了。

下面还有一条语句，创建全局变量`neoformat_enabled_python`，表示针对 Python 启用的格式化工具，这里我们使用上面创建变量后注册的`black`。

#### 3.3 执行格式化
加了上面的 VimScript 配置后，我们在编辑文件时，就可以使用 `:Neoformat` 命令来格式化代码了。

如果想要使用特定的格式化工具，可以使用`:Neoformat <formater>` 来操作。

#### 3.4 保存文件时自动格式化
前面的配置我们还需要手动执行`:Neoformat` 命令来格式化，下面我们添加一些配置到`~/.config/nvim/init.vim`，在保存文件时自动地进行格式化。

```vimscript
augroup fmt
  autocmd!
  autocmd BufWritePre * Neoformat
augroup END
```

这段代码创建了一个自动化组并命名为`fmt`，用于将一组命令放在一起，方便管理。

我们首先使用`autocmd!`清空这个自动化组中的所有自动化命令，避免影响后面的命令设置。

然后用`autocmd BufWritePre * Neoformat`来完成在写buffer之前，对所有类型的文件都执行`Neoformat`命令。`autocmd`表示这是一条自动化命令。`BufWritePre`表示是在Write Buffer之前执行的操作,`*`表示匹配任意的文件，如果是`*.py`则只匹配后缀为`.py`的文件。`Neoformat` 表示要执行的命令。

这样，在保存文件时，就可以自动执行代码格式化了。


#### 3.5 调试命令
如果出现格式化错误，或者格式化不生效，可以设置 `:set verbose=1` 来打开 NeoVim 的 log 显示，查看报错信息。实际测试发现这个命令真的很有用，很多信息打印出来后，对于定位问题帮助很大。


### 4. neoformat 格式化 C/C++ 代码
对 C/C++代码的格式化与 Python 是类似的，只不过使用的格式化工具不同而已。这里以 [clang-format](https://clang.llvm.org/docs/ClangFormat.html) 为例，记录需要执行的步骤。

#### 4.1 安装格式化工具
Ubuntu:
```bash
sudo apt install clang-format
```

Mac:
```bash
brew install clang-format
```

#### 4.2 格式化配置
```vimscript
let g:neoformat_c_clangformat = {
            \ 'exe': 'clang-format',
			\ 'args': ['-assume-filename=%:p'],
            \ 'stdin': 1,
            \ }

let g:neoformat_enabled_c = ['clangformat']
```
与 Python black 的配置类似，语言修改为`c`, formatter 修改为 `clangformat`，参数有所不同，`-assume-filename=%:p` 表示将当前编辑的文件名传递给 clang-format，以便它可以正确地处理预编译指令等特殊情况。

#### 4.3 自定义格式化文件
如果不想用默认的 clang-format 格式化配置，可以通过下面的方式来生成格式化文件，并通过`args` 参数传递给Neoformat来使用。

首先生成一个默认的配置文件，例如选择以google的风格来生成:
```bash
clang-format -style=google -dump-config > /Users/name/.clang-format
```
然后编辑生成的文件，修改为你想要的格式。例如我想修改默认的2空格缩进为4空格，那么去掉默认文件中的`# BasedOnStyle:  Google`的注释，继承google风格的默认配置，删除后面所有的内容，只修改`IndentWidth` 项：
```plain
---
Language:        Cpp
BasedOnStyle:  Google
IndentWidth:     4
```
然后用`--style=/path/to/.clang-format`来代码规范文件：
```vimscript
let g:neoformat_c_clangformat = {
            \ 'exe': 'clang-format',
			\ 'args': ['-assume-filename=%:p', '--styel=/Users/name/.clang-format'],
            \ 'stdin': 1,
            \ }

let g:neoformat_enabled_c = ['clangformat']
```
#### 4.4 保存文件时自动格式化
上面 3.4 部分的代码已经开启了保存时自动格式化代码，这里不需要额外增加配置了。


### 5. 总结
总结下来，涉及到的需要增加在`~/.config/nvim/init.vim`中的代码如下：
```vimscript
call plug#begin("~/.nvim/bundle")
...
" 增加neoformat
Plug 'sbdchd/neoformat'
...
call plug#end()

...

" code format
augroup fmt
  autocmd!
"  autocmd BufWritePre * undojoin | Neoformat
  autocmd BufWritePre * Neoformat
augroup END

" format python
let g:neoformat_python_black = {
            \ 'exe': 'black',
            \ 'args': ['-q', '-'],
            \ 'stdin': 1,
            \ }

let g:neoformat_enabled_python = ['black']


" format c/c++
let g:neoformat_c_clangformat = {
            \ 'exe': 'clang-format',
			\ 'args': ['-assume-filename=%:p'],
            \ 'stdin': 1,
            \ }

let g:neoformat_enabled_c = ['clangformat']
```

格式化工具需要单独通过命令行来安装:
```bash
python3 -m pip install black
brew install clang-format
```

通过 `:set verbose=1` 来打开 log 信息，对于定位问题很有帮助。

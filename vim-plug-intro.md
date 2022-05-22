---
title: vim-plug：简洁高效的Vim插件管理工具
date: 2019-03-12 22:20:26
tags:
 - Linux
 - Vim
 - 总结
 - NeoVim
---

今天无意中发现了这个[vim-plug](https://github.com/junegunn/vim-plug)这个简洁又高效的Vim插件管理工具，试了下，安装插件简直没法再容易，大大减小了配置难度，对于我这种既想要Vim及插件强大的功能但又不想花费太多时间到配置上的懒人来说，Vim-plug简直就是神器了。  
借用作者的原话，Vim-plugin有下面的优点：
> 1. Easier to setup: Single file. No boilerplate code required.
> 2. Easier to use: Concise, intuitive syntax
> 3. Super-fast parallel installation/update (with any of +job, +python, +python3, +ruby, or Neovim)
> 4. Creates shallow clones to minimize disk space usage and download time
> 5. On-demand loading for faster startup time
> 6. Can review and rollback updates
> 7. Branch/tag/commit support
> 8. Post-update hooks
> 9. Support for externally managed plugins

既然配置又简单，功能又强大，为甚不小小折腾一番，提高工作效率呢？下面我以在Vim中安装一个Python的检查器为例，对Vim-plugin的使用进行说明。
<!--more-->
实验环境：
 1. Ubuntu 16.04
 2. Python 3.5
 3. Vim 8.1.946

别的操作系统和Vim版本（如NeoVim）的教程请参考[官方文档](https://github.com/junegunn/vim-plug)。

## 1. 安装
安装vim-plug很简单，下载[plugin.vim](https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim)到`~/.vim/autoload`目录即可，可以使用下面的一行命令来下载：
```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

## 2. 使用
vim-plu通过在`~/.vimrc`中增加`call plugin#begin()`和`call plugin#end()`来定义和管理插件，插件以`Plug 'github_url'`形式来描述, 格式如下：
```vim
call plug#begin()
# 完整的GitHub仓库地址
Plug 'https://github.com/junegunn/vim-github-dashboard.git'
# 简写形式，只写username/repo即可
Plug 'junegunn/fzf'
call plug#end()
```
配置文件写好后，重新打开Vim,在命令模式下输入`:PlugInstall`即可安装配置文件中设置的插件。  
就这样，大功告成了，下面要做的就是探索各种强大或神奇的Vim插件了，这个可以通过搜索引擎搜索或在GitHub搜索或者找博客和文章发现自己想要的。

下面可以[ALE](https://github.com/w0rp/ale)这个代码检查工具为例，进行vim-plug的简单示范。

## Vim-plug 安装ALE对Python代码进行检查
找到ALE的GitHub地址：`https://github.com/w0rp/ale`，以简写形式加入到`.vimrc`中：
```vim
call plug#begin()
Plug 'W0rp/ale'
call plug#end()

let g:ale_linters = {'python': ['flake8']}
```
最后的`let`语句是Vimscript的语法，这行表示对Python的代码检查工具，我们采用flake8这个Python包，因此需要用pip安装下：
```bash
pip3 install --user flake8
```
保存`.vimrc`后，打开Vim，在命令模式下，输入`:PlugInstall`，会出现一个新panel，上面显示安装的输出，包括从GitHub下载文件等等，如果发现比较慢，请多等会…  
等安装完成后，打开一个Python文件，你那不符合规范的地方就被标出来了，光标移动到提示的那行，底下会出现报错的原因，可以根据提示信息修改你的代码，下面是一个效果图：
![ALE报错信息](/imgs/vim-plug-ale.png)
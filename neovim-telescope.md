---
title: neovim telescope 插件简要教程
date: 2023-03-28 23:44:37
tags:
- Vim
- NeoVim
- Linux
---
## 1. 概述
[telescope](https://github.com/nvim-telescope/telescope.nvim/) 是一款强大的 neovim 插件，可以在 neovim 中提供文件名搜索和文本内容搜索的功能，以及更多复杂的功能，具体的show case可以看[这里](https://github.com/nvim-telescope/telescope.nvim/wiki/Showcase)。我安装 telescope 主要是想利用它在大型项目中的文件名搜索和文本内容搜索能力，这里记录一下安装流程和使用概要。

<!--more-->

## 2. 安装
首先需要安装 neovim。具体步骤可以看[这里](https://github.com/neovim/neovim/wiki/Installing-Neovim)。

注意 telescope 需要nvim 0.7.0及以后的版本，因此如果你neovim 版本本身比较低的话，需要升级。

安装 neovim 后还需要进行配置。我的 neovim 配置是复制的这个[仓库](https://github.com/bigeagle/neovim-config)，按照README来进行操作，可以快速地安装好，这里不赘述。


telescope 支持多种插件系统，我使用的 vim-plug，在`~/.config/nvim/init.vim` 添加下面两行：
```vim
Plug 'nvim-lua/plenary.nvim'
Plug 'nvim-telescope/telescope.nvim', { 'tag': '0.1.1' }
```
然后在nvim中输入`:PlugInstall` 来安装插件。

由于插件是在GitHub上下载的，有时候可能安装会卡住，需要多尝试几次，即多次执行`:PlugInstall`命令。

安装完成后，执行`:Telescope find_files`来验证安装是否正确。如果能弹出输入框，说明安装成功了。

这个命令用来模糊匹配当前目录下的所有文件名，对于快速切换编辑文件非常方便。

## 3. `live_grep` 功能
除了`find_files`命令，`live_grep`也是一个很有用的命令，可以快速搜索某些代码，把含搜索代码的文件打开。

这个功能需要依赖[ripgrep](https://github.com/BurntSushi/ripgrep)，因此要先安装它，具体安装命令如下：
```bash
# mac
brew install ripgrep

# debian/ubuntu
sudo apt-get install ripgrep

# arch
pacman -S ripgrep

# centos
sudo yum-config-manager --add-repo=https://copr.fedorainfracloud.org/coprs/carlwgeorge/ripgrep/repo/epel-7/carlwgeorge-ripgrep-epel-7.repo
sudo yum install ripgrep

# windows 
scoop install ripgrep
```
安装完后在命令行输入`ag -h` 验证安装是否成功。

ag 安装完成后，在nvim输入`:Telescope live_grep` 就可以搜索你想要的代码了。

## 4. 快捷键
上面的两个常用功能输入都比较繁琐，有没有什么快捷键可以快速打开呢？是有的，官方GitHub给出了几行代码，加入到`~/.config/nvim/init.vim`的最后：
```vim
" Find files using Telescope command-line sugar.
nnoremap <leader>ff <cmd>Telescope find_files<cr>
nnoremap <leader>fg <cmd>Telescope live_grep<cr>
nnoremap <leader>fb <cmd>Telescope buffers<cr>
nnoremap <leader>fh <cmd>Telescope help_tags<cr>

" Using Lua functions
nnoremap <leader>ff <cmd>lua require('telescope.builtin').find_files()<cr>
nnoremap <leader>fg <cmd>lua require('telescope.builtin').live_grep()<cr>
nnoremap <leader>fb <cmd>lua require('telescope.builtin').buffers()<cr>
nnoremap <leader>fh <cmd>lua require('telescope.builtin').help_tags()<cr>
```

然后在Normal模式输入`\ff`就可以打开`find_files`命令窗口，输入`\fg`就可以打开`live_grep`窗口了。

更多详细命令和功能参见GitHub 页面。

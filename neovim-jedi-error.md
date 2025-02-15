---
title: neovim/vim 中遇到jedi-vim 插件报错解决
date: 2021-11-06 14:45:03
tags:
- Python
- NeoVim
- 总结
- 错误汇总
---

## 问题描述
[jedi-vim](https://github.com/davidhalter/jedi-vim)是vim/neovim的Python代码自动补全插件，很好用，不过最近遇到这样一个问题，用neovim 打开python文件时，会有这样的提示：

```bash
Error: jedi-vim failed to initialize Python: jedi#setup_python_imports: ImportError: bad magic number in 'jedi.common':
```
这里记录一下解决办法.

<!--more-->

## 解决办法
这个问题可能是更新`jedi-vim`插件时, 缓存的`.pyc` 文件没删除导致的，因此我们找到插件目录，手动删除这种类型的文件就行：
```bash
# 如果使用的是vim，将下面路径中的~/.nvim 替换为~/.vim
cd ~/.nvim/bundle/jedi-vim
find . -type f -name "*.pyc" -exec rm {} \;
```

## 参考
1. <https://github.com/davidhalter/jedi-vim/issues/1026>

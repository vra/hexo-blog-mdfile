---
title: Neovim conceal机制导致markdown语法隐藏的问题
date: 2025-02-15 16:40:47
tags:
- Vim
- NeoVim
- Markdown
---

`conceal` 是 Vim/Neovim 中一个用于**优化显示效果**的机制，它可以将某些语法符号替换为更简洁的视觉表示（或完全隐藏）。这在 Markdown、LaTeX 等格式中常用于提升可读性，但有一个问题：不太好确定自己的markdown标签是否写完了，因此markdown文件可能最后少了一个\`\`\`，渲染结果出错。而对于我来说，我不需要在编辑器里面看到最终的渲染效果，所以这个功能完全可以去掉。

为了确定是插件导致的还是Neovim自带的功能，我用下面的命令打开不启用所有插件的Neovim:
```bash
nvim -u NORC /path/to/file
```
结果markdown渲染正常，因此确认问题是由某个插件引入的。

然后可以通过一个个启用插件的方式，来验证是哪个插件导致的问题，但由于插件太多，太麻烦，我问了DeepSeek R1，验证下面的语句可以解决这个问题：
```bash
let g:vim_markdown_conceal = 0
let g:vim_markdown_conceal_code_blocks = 0
```
也有人说是`indentLine`设置导致的，但没有再验证。

也是通过这次搜索，了解了conceal这个术语。

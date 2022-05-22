---
title: my-dot-file
date: 2019-05-07 16:58:13
tags:
 - 总计
 - Linux
---

## oh-my-zsh
```bash
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

## 修改SHELL为zsh
```bash
sudo usermod -s /bin/zsh $(whoami)
```

## 安装fzf
```bash
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

## git alias设置
```bash
git config --global alias.st status
git config --global alias.a add  
git config --global alias.p push 
git config --global alias.pu pull
git config --global alias.ci commit
git config --global alias.br branch
git config --global alias.unstage 'reset HEAD'
git config --global alias.last 'log -1'
git config --global alias.co checkout
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```
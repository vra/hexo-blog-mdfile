---
title: homebrew禁止执行install命令时自动更新
date: 2023-06-08 22:02:49
tags:
- Homebrew
- macOS
---
[Homebrew](https://brew.sh/) 是 macOS 下的默认的包管理器，不需要sudo权限就可以安装包，比较好用。

不过用`brew install`安装包时有个问题，它默认会先执行`brew update`来更新brew的版本。但由于brew 的源国内访问比较慢，常常`brew update`执行耗时比较久，影响每次安装包的体验。

解决办法是设置`HOMEBREW_NO_AUTO_UPDATE`环境变量为1，这样每次`brew install`时跳过更新brew的步骤，实际体验安装包速度提升明显。

可以添加下面的语句到你的.bashrc或.zshrc中，重启shell即生效:
```bash
export HOMEBREW_NO_AUTO_UPDATE=1
```

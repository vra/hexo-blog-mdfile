---
title: GitPod简单使用说明
date: 2025-02-15 16:22:38
tags:
- GitPod
- 网站
- App
---
GitPod是一个云端开发IDE，可以访问`gitpod.io`，绑定GitHub账号后打开GitHub上的任意项目，也可以通过安装浏览器插件，直接在GitHub网站打开IDE。

GitPod打开后默认是个VS Code在线环境，有一台国外的容器可以使用，机器配置如下：

+ CPU: 16核，型号AMD EPYC 7B13
+ 内存：64G
+ 存储：30G

由于它的服务器在国外，因此可以快速下载GitHub, Google Drive或Hugging Face上的一些模型，然后用Python开一个简单的网页服务(`python -m http.server`)，再在本地用wget下载模型，速度还可以。

GitPod主打的一个点是快速启动开发环境，可以通过在https://gitpod.io/user/preferences 设置中指定dotfile来设置启动环境

这个dotfiles仓库可以保存你常用的rc文件等，保证熟悉的环境能够快速上手，例如我将自己的常用配置放到<https://github.com/vra/dotfiles>，开机就能用上熟悉的开发环境了。

总之，GitPod可以作为一个免费的临时服务器和在线IDE，偶尔用用还不错。

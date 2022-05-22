---
title: Windows Subsystem for Linux 切换网络脚本
date: 2020-03-04 21:19:36
tags:
- Windows
- Linux
- WSL
- Trick
---
在启用/关闭 VPN 的时候， WSL 里面，有时候网络会无法连接。根据[这里](https://github.com/microsoft/WSL/issues/416)的讨论，这是由于 WSL 在网络变化的时候，未能正确解析 DNS 导致的。网友在这里也给出了一个解决办法，试了以后是可以的，因此记录下来。
<!--more-->
在 WSL 的命令行执行下面的命令：
```bash
cd ~
wget https://gist.github.com/matthiassb/9c8162d2564777a70e3ae3cbee7d2e95/raw/b204a9faa2b4c8d58df283ddc356086333e43408/dns-sync.sh 
sudo mv dns-sync.sh /etc/init.d/dns-sync.sh
sudo chmod +x /etc/init.d/dns-sync.sh
unlink /etc/resolv.conf
```
如果 Gist 访问不了的话，可以从[这里](https://raw.githubusercontent.com/vra/wsl-dns-sync/master/dns-sync.sh)下载我fork的一个版本。详细讨论可以参考[这里](https://gist.github.com/matthiassb/9c8162d2564777a70e3ae3cbee7d2e95)的讨论。

注意：每次切换网络时需要手动执行最后的一句命令：
```bash
sudo service dns-sync.sh start
```
参考：
1. <https://gist.github.com/matthiassb/9c8162d2564777a70e3ae3cbee7d2e95>
2. <https://github.com/microsoft/WSL/issues/416>

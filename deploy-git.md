title: 搭建自己的Git服务器
date: 2017-04-19 21:58:08
tags:
 - Git
 - Linux
---
相信很多人都对GitHub和GitLab很熟悉了，这些基于Git版本控制的在线代码托管平台由于丰富的内容，简洁的操作和集成一体化以及风靡全球了。今天我好奇，想了解下如何搭建自己的Git服务器，于是查了一些资料，记录下整个的流程。
![](/imgs/git_logo.png)
<!-- more-->
### 为什么要用自己的Git服务器？
想了想，有下面的优势：
 1. 免费的私有仓库
 2. 完全的对项目的控制
 3. 了解GitHub和GitLab等背后的运作原理

### 准备
1. 一台可以通过域名或网址访问的服务器
2. 服务器上安装有ssh, git等工具，可以通过下面命令来安装:
```bash
sudo apt-get install  openssh git
```

### 创建git用户
为了访问的便捷，我们使用git用户的身份来创建代码仓库，当然任何用户都是可以的，只不过在git clone的时候，将git@server改成别的用户名即可。
```bash
sudo adduser git
```

### 上传公钥
为了git clone 仓库的时候免去输入git用户密码的烦恼，我们这里发送客户端的用户的ssh公钥到git用户的`~/.ssh/authorized_keys`文件，具体执行下面这条命令即可:
```bash
ssh-copy-id -i git@114.215.66.43
```

### 修改git用户的登录权限
因为git用户是专门用来上传代码的，所以禁用git用户的登录权限，将git用户的登录shell改为`/usr/bin/git-shell`
```bash
sudo vi /etc/passwd 
# change shell of git to /usr/bin/git-shell
```

### 创建裸仓库
因为git仓库不需要再服务器上更新，而是通过远程push进行更新，所以我们建立一个裸仓库即可，裸仓库即没有项目代码而只有git元数据的仓库，注意裸仓库后缀都是git。
```bash
su -l git
mkdir -p ~/src/my-repo.git
git init --bare my-repo.git
```
这样服务器端的操作就完成了。
### 客户端操作
客户端就按正常的git 操作来克隆刚才创建的仓库：
```bash
git clone git@114.215.66.43:/home/git/src/my-repo.git
```
后面就跟正常的操作完全一样了，演示一个简单的例子：
```bash
cd my-repo
echo README >> README
git add README
git commit -m "add README"
git push origin master
```

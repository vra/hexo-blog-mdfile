---
title: fly.io 使用指南
date: 2022-10-06 08:18:04
tags:
 - fly.io
 - Docker
 - Linux

---
### 1. 概述
最近看技术论坛，发现提到 [fly.io](https://fly.io/) 的次数越来越多了。 fly.io 是一个容器化的部署平台，只需要一个`Dockerfile`文件就能部署代码到fly.io 的服务器上，同时还自动生成域名。其他的好处多多，我根据自己体验，我总结成了下面的这些条：

+ ~~有免费使用的额度。不填写信用卡信息可以创建一个App，完全不收费；填写信用卡信息后每月有一定额度的免费流量，超过额度会额外收费。所以想做个小demo完全可以不填信用卡试用。~~ 目前已经不支持无信用卡使用了，参见[这里](https://community.fly.io/t/is-it-free-getting-error-we-need-your-payment-information-to-continue/8871)的讨论

+ 自动生成域名。比如你创建一个名字叫`my_demo`的App，那么部署完成后，就会生成`my_demo.fly.dev`的域名，可以全球访问，不用自己单独买域名了。

+ 可以 SSH 连接进入服务器。部署完成后，可以通过`flyctl ssh console` 命令登录部署的服务器，所以相当于你有了一台免费的VPS，可以做你想做的任何事情。

+ 部署简单，采用`flyctl` 命令集合统一部署;支持各种语言的各种框架来搭建部署环境，能自动识别当前目录下代码所采用的是哪个框架，自动部署。

下面简单记录一下使用的流程和一些教程里面没提及的使用命令。
<!--more-->

### 2. 部署一个应用
这里以Python 的 Flask 框架为例，进行部署的步骤总结，其实fly.io支持很多框架，可以在[这里](https://fly.io/docs/speedrun/)查看。

#### 2.1 安装 flyctl
首先需要安装 flyctl 这个工具：
Mac:
```bash
brew install flyctl
```
Linux:
```bash
curl -L https://fly.io/install.sh | sh
```
Windows:
在Powershell中运行下面的命令:
```bash
iwr https://fly.io/install.ps1 -useb | iex
```
如果执行`flyctl version` 不报错，就说明安装成功了。

**一个小技巧，flyctl还有个alias fly，敲起来更简短些。**

安装这个工具是一次性的，后面不需要再操作

#### 2.2 创建并登录账号
创建账号:
```bash
fly auth signup
```
会打开网页，选择自己要创建账号的方式，GitHub账号或者邮箱等。

创建完成后登录账号:
```bash
fly auth login
```

#### 2.3 先在本地将Flask demo跑起来
这里采用 fly.io 提供的Flask demo 代码，先在本地跑起来:
```bash
git clone https://github.com/fly-apps/python-hellofly-flask
cd python-hellofly-flask
python -m venv flask-env
source flask-env/bin/activate
python -m pip install -r requirements.txt
FLASK_APP=hellofly flask run
```
然后访问`http://127.0.0.1:5000` 就能看到网站，说明本地搭建成功了。

### 2.4 部署到 fly.io
在当前目录下，执行`fly launch`，进入交互式界面创建App:
```bash
flyctl launch
Creating app in /Users/username/project/demo/flyio_demo/python-hellofly-flask
Scanning source code
Detected a Python app
Using the following build configuration:
        Builder: paketobuildpacks/builder:base
? Overwrite "/Users/username/project/demo/flyio_demo/python-hellofly-flask/Procfile"? No
? App Name (leave blank to use an auto-generated name): treehole
Automatically selected personal organization: username
? Select region: hkg (Hong Kong, Hong Kong)
Created app treehole in organization personal
Wrote config file fly.toml
? Would you like to set up a Postgresql database now? No
We have generated a simple Procfile for you. Modify it to fit your needs and run "fly deploy" to deploy your application.
```
然后执行`flyctl deploy` 来将Appb部署到 fly.io 的服务器上:
```bash
flyctl deploy
```

执行成功后，可以用`flyctl open`来打开浏览器，访问自己部署的App，网址是`appname.fly.dev`。

如果后面有源码或者配置的修改，可以多次执行`flyctl deploy`，会生成新的版本v0，v1, v2依次往下，往fly.io上部署。

接下来就是修改你的Flask源代码，完成更复杂有真正意义的功能了。

#### 2.5 别的有用的flyctl 命令
+ 查看App状态: `flyctl status`
+ 查看App信息: `flyctl info`
+ 查看App列表: `flyctl apps list`
+ 查看App的IP: `flyctl ips list`
+ 销毁某个App: `flyctl apps destroy <appname>`


### 3. 登录部署机器
机器部署完成后，可以通过`flyctl ssh console`来登录机器，登录后就跟普通Linux机器的使用是一样的了，可以随意探索。


### 4. 复制部署机器上的文件到本地
在一个终端输入下面的命令来代理端口
```bash
fly proxy 10022:22
```
然后保持上面的终端打开，在另一个终端输入下面的命令:
```bash
scp -P 10022 root@localhost:/path/of/file/on/vm  /path/on/local
```
修改文件的路径就能将文件复制过来


### 5.一点感想
当demo部署服务成功后，却不知道能做什么真正有意义的事情，或许缺少的不是工具，而是真正产生价值的点子。

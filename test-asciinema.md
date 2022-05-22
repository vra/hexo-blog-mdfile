title: 在Hexo博客里面插入asciinema终端记录视频
date: 2016-11-14 15:33:47
tags:
 - Linux
 - asciinema
---
## 概述
前几天发现了一个很有意思的记录终端操作的工具[asciinema](https://asciinema.org/),使用起来异常简单功能却很强大，很佩服开发者的想象力和创造力。  
今天我在想，能否在Hexo博客里面插入asciinema录的视频呢？Google了一下，发现真的已经有人做出了该功能的插件[hexo-tag-asciinema](https://github.com/narongdejsrn/hexo-tag-asciinema),安装了下果然可以在博客里面插入asciinema，而且一个超级简单的命令即可完成。像下面就是一个例子(用C++编写一个简单的HelloWorld程序)：
{% asciinema 92655 %}

下面详细介绍每个步骤。
<!--more-->

## asciinema安装
参照[这里](https://asciinema.org/docs/installation)的教程,常见的asciinema的安装方式有下面2种：
 1. 通过系统的包管理软件安装
 Debian:
```bash
sudo apt-get install asciinema
```
 Ubuntu:
```bash
sudo apt-add-repository ppa:zanchey/asciinema
sudo apt-get update
sudo apt-get install asciinema
```

2.通过pip3安装，需要先安装python3
```bash
sudo pip3 install asciinema
```

## asciinema使用
安装好后，打开终端,输入`asciinema rec` 开始记录，按`Ctrl-D`结束记录。  
结束记录后，会让你选择是否需要上传数据，如果选择`Y`,则会给出一个URL，点击该URL即可访问你刚才录的视频。
另外，你也可以在asciinema官网上注册帐号，这样你所有记录的数据都可以保存在上面，你可以通过`asciinema auth`来验证帐号。

## 在Hexo里面插入asciinema的视频
假设你已经在本地安装好了Hexo博客系统而且已经通过asciinema录制好了视频**并上传到asciinema网站上**。  
首先是通过`npm`安装[hexo-tag-asciinema](https://github.com/narongdejsrn/hexo-tag-asciinema):
```bash
npm install --save hexo-tag-asciinema 
```
然后在Hexo博客的`markdown`文件里面，使用下面的命令来插入视频:
```js
{% asciinema video_id %}
```
其中`video_id`是你上传的视频的编号，比如你视频所在的页面是`https://asciinema.org/a/41100`, 那在`video_id`那里填`41100`。
然后保存markdown文件，执行`npm install`安装必要的包，再`hexo deploy`部署你的博客，就可以看到效果了。

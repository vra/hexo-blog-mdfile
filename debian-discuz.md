title: 'Debian 下搭建Discuz!论坛'
id: 88
categories:
  - 学习总结
date: 2014-12-14 15:22:52
tags:
  - Linux
---

[Discuz!](http://www.discuz.net/)是一个用PHP编写的论坛框架,像[六维](http://bt.neu6.edu.cn/ "六维")以及我们学校少年班学院的[格物致知论坛](https://gewu.ustc.edu.cn)都是搭建在Discuz!上面的,看论坛页面左下角,都写着"Powered by Discuz!_xxx",_其中xxx表示Discuz!的版本号.因为我们实验室师兄用内网搭建了个服务器管理的论坛,而且我之前也尝试过搭建wordpress博客(详细过程可以看我[这篇博客](https://vra.blog.ustc.edu.cn/debian-wordpress/))而且成功了(其实没什么技术含量....),估计这个过程也差不多,所以我就想试试Discuz!能否搞定.但我们学校有规定,不能用freeshell搭建网络论坛的,所以我就在我电脑上试着搞搞Discuz!玩玩.
<!--more-->

![](/uploads/2014/12/Selection_0201.png)

整个过程大概分为两部分,第一部分就是搭建LAMP整个框架,第二部分就是在LAMP基础上配置Discuz!.

其实LAMP框架是最核心的东西,有了这个框架,其实我们完全不用什么wordpress和Discuz!,只要你可以写后端的PHP程序和前端的HTML,CSS,JS这些代码,完全可以自己写网站或论坛等.而wordpress,Discuz!给了不会或者写不好代码的人一个简易的搭博客和搭论坛的方式,大大简化了步骤,缩短了开发时间.

搭建LAMP的过程我已经在[搭wordpress的博客](https://vra.blog.ustc.edu.cn/debian-wordpress/)里面写了,也可以访问[debian的wiki.](https://wiki.debian.org/LaMp)下面我着重陈述配置Discuz!的部分.

1.下载Discuz!压缩文件:

下载地址为:[http://www.discuz.net/thread-3570835-1-1.html](http://www.discuz.net/thread-3570835-1-1.html).有简体和繁体的GBK和UTF8版本,可以根据自己需要下载相应版本.

2.将下载的Discuz!压缩文件解压:

```bash
unzip /path/to/Discuz_XXX_XX_XXX.zip
```

其中/path/to/要改为到压缩包的路径,Discuz_XXX_XX_XXX.zip要改为你下载的压缩包的名字.

3.将解压后的upload文件夹复制到apache2的默认网页目录(/var/www/)下的forum下:

```bash
mkdir /var/www/forum
cp -R upload /var/www/forum
```

4.修改forum目录下的子目录的访问权限:

```bash
cd /var/www/forum
sudo chmod -R 777 config data uc_client uc_server
```

经过简单的几步操作,我们就可以开始Discuz!的配置了.

5.Discuz!数据库配置:

在浏览器中输入http://localhost/forum,就会出现Discuz!的配置页面:

![](/uploads/2014/12/Selection_013.png)

然后我们一步一步来安装.

a.首先在这个页面选择我同意按钮,就会到1.开始安装页面:

![](/uploads/2014/12/Selection_014.png)

b.如果之前修改了forum子目录的权限的话,这一步是没问题的.如果有问题,请检查你的chmod那个命令执行了没有.没问题的话,按页面底部的下一步按钮,就到2.设置运行环境页面:

![](/uploads/2014/12/Selection_015.png)

这一步选择默认即可.下一步就到了3.安装数据库页面了:

![](/uploads/2014/12/Selection_016.png)

这一步就是配置数据库,设置管理员信息.要注意的是管理员密码是必须填的,也是管理员登录这个论坛的passwd.填好之后下一步,就到了4.安装数据库:

![](/uploads/2014/12/Selection_017.png)

可以看到,这一步就是执行上一步表中所填的内容,即在MySQL数据库中创建数据库,创建表格,执行初始化操作等等.安装完成后就到了这个页面:

![](/uploads/2014/12/Selection_018.png)

看到右下角一行小字:"您的论坛已安装完成,点此访问"了吗?,点击这个按钮,就可以看到你的论坛了!

下面是我发了一个帖子的页面:

![](/uploads/2014/12/Selection_019.png)

至此,Discuz!搭建就完成了.

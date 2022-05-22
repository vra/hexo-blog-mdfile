title: 在Debian下搭建基于Apache-Php-MySQL的wordpress博客
id: 77
categories:
  - 学习总结
date: 2014-12-13 22:50:50
tags:
  - Linux
  - Wordpress
---

wordpress是一个流行的博客搭建框架,为不会html,css和js的人提供了搭建博客的便捷方式.我这里是在我的笔记本上搭建了一个wordpress博客,这里把详细的搭建过程写出来.
<!--more-->
我的系统信息如下:

[![Selection_001](/uploads/2014/12/Selection_001.png)](/uploads/2014/12/Selection_001.png)

具体的操作过程如下描述.

1.安装apache2服务器

[![Selection_003](/uploads/2014/12/Selection_003.png)](/uploads/2014/12/Selection_003.png)

其中apache2-doc是apache服务器的说明和配置文件,libapache2-mod-php5是apache的php模块库文件.

安装成功后,重启apache2服务器,

[![Selection_001](/uploads/2014/12/Selection_0012.png)](/uploads/2014/12/Selection_0012.png)

此时在浏览器地址栏里面输入http://localhost,则会看到如下的页面,提示我们apache2服务器已经安装成功.

&nbsp;

[![Selection_002](/uploads/2014/12/Selection_0021.png)](/uploads/2014/12/Selection_0021.png)

2.关于apache2的配置信息:

a.apache2的配置文件目录是/etc/apache2.在debian下,配置文件被打散分到了该目录下的几个子文件夹中.可以看该目录下的文件:

[![Selection_002](/uploads/2014/12/Selection_002.png)](/uploads/2014/12/Selection_002.png)

其中apache2.conf 是主配置文件,该目录下还有ports.conf文件用来配置服务器的监听端口.此外mod-enabled和sites-enabled和conf-enabled子目录下分别有一个.conf文件,详细的配置说明可以看相应的说明.

b.apache2安装时会创建一个叫做www-data的用户,所有apache相关的进程都由该用户来启动执行.可以在浏览器里面访问localhost的时候,用top命令查看:

[![Selection_003](/uploads/2014/12/Selection_0031.png)](/uploads/2014/12/Selection_0031.png)

上图中第5条记录即为apache2服务器的进程开销情况.

c.apache2的默认网页和脚本存放目录为/var/www/html,在该目录下存放的网页(除了index页面)都可以通过http://localhost/filename访问到,如该目录下有个aboutme.html,则可以在浏览器里输入http://localhost/aboutme.html访问.

3.安装php:

[![Selection_007](/uploads/2014/12/Selection_007.png)](/uploads/2014/12/Selection_007.png)

其中php5-mysql是php和mysql数据库的接口,为了使用mysql数据库必须安装这个包.

安装完成后,可以通过如下方法检查php的安装是否成功:

a.在/var/www/html目录下,编写如下内容的文件phpinfo.php:
<pre>&lt;?php phpinfo(); ?&gt;</pre>
然后在浏览器中访问该页面:http://localhost/phpinfo.php,如果出现如下页面,则说明php安装已经成功.

[![Selection_008](/uploads/2014/12/Selection_008.png)](/uploads/2014/12/Selection_008.png)

往下拉一下网页右侧滚动条,就可以看到下面是php支持的各个模块和组件.看起来相当多.

4.安装mysql:

[![Selection_009](/uploads/2014/12/Selection_009.png)](/uploads/2014/12/Selection_009.png)

安装完成后,刷新刚才的phpinfo页面,往下拉到中间位置的时候,可以看到mysql和mysqli,说明msyql也已经安装成功了.

[![Selection_010](/uploads/2014/12/Selection_010.png)](/uploads/2014/12/Selection_010.png)

&nbsp;

5.下载wordpress压缩文件:

访问http://cn.wordpress.org,如下图所示.在右侧中间位置有压缩包供下载,点击下载即可.

[![Selection_011](/uploads/2014/12/Selection_011.png)](/uploads/2014/12/Selection_011.png)

也可复制该链接地址,用wget下载(感觉好像用wget下载比较快):

[![Selection_012](/uploads/2014/12/Selection_012.png)](/uploads/2014/12/Selection_012.png)

下载后解压该文件:

[![Selection_013](/uploads/2014/12/Selection_0131.png)](/uploads/2014/12/Selection_0131.png)

解压后的文件放在wordpress文件夹下,可以看看里面的内容:

[![Selection_004](/uploads/2014/12/Selection_0041.png)](/uploads/2014/12/Selection_0041.png)

可以看到大多都是以wp开头的文件或文件夹,这些文件夹保存了配置博客的脚本和展示给访问者的页面框架,而其他的信息则保存在数据库中.

因为我们默认的网页存放目录是/var/www/html,所以要将该文件夹内文件移动到该目录下才生效,所以执行如下移动操作:
<pre>mv -R wordpress /var/www/html</pre>
该操作会用wordpress目录替换原来的html目录.

现在在浏览器中打开http://localhost,就会看到开始wordpress的配置的页面了:

[![Selection_016](/uploads/2014/12/Selection_0161.png)](/uploads/2014/12/Selection_0161.png)

然后按照步提示,在mysql创建相应的wordpress数据库,整个博客就算搭建完成了!

下面是我搭建的博客(随便从网上抄了点内容...):

[![Selection_017](/uploads/2014/12/Selection_0171.png)](/uploads/2014/12/Selection_0171.png)

(-完-)

title: 在Apache上部署Django项目
date: 2016-07-19 16:44:59
tags:
 - Python
 - Django
 - Apache
 - Web编程
---
###0.概述
Django是一个基于Python的web开发框架，在实际生产环境中部署的时候，还需要用Apache容器来部署。这里记录下如何在Debian系统中用Aapche和[mod_wsgi模块](https://pypi.python.org/pypi/mod_wsgi)来部署Django项目。
<!--more-->
###1.系统信息
```bash
$ uname -a  
Linux iZ284ov0vfwZ 3.2.0-4-amd64 #1 SMP Debian 3.2.81-1 x86_64 GNU/Linux  
$ lsb_release -a  
No LSB modules are available.  
Distributor ID: Debian  
Description:    Debian GNU/Linux 7.11 (wheezy) 
Release:        7.11  
Codename:       wheezy  
$ sudo apachectl -v  
Server version: Apache/2.2.22 (Debian)  
Server built:   Aug 18 2015 09:49:50  
```
**我用的是Debian发行版，Apache的配置与别的发行版有较大不同，这里以Debian为例进行说明，别的发行版需要进行一定的修改。**
###2. 安装Django和Apache
Django可以通过如下命令安装:
```bash
sudo pip install Django==1.9.0 #设置版本号为1.9.0
 ```
 Apache通过不同发行版的包管理命令安装。在debian下，是:
 ```bash
 sudo apt-get install apache2
 ```
###3. 安装mod_wsgi模块
mod_wsgi可以通过pip安装，但是需要提前在系统安装`apache-dev`包，但是在Debian发行版上，这个包名叫`apache2-prefork-dev`，详情参考[这里](http://stackoverflow.com/a/16869017/2932001)。通过如下命令安装
```bash 
sudo apt-get install apache2-prefork-dev
```
然后pip 安装mod_wsgi:
```bash
sudo pip install mod_wsgi
```
此外也可以自己编译mod_wsgi：首先从[这里](https://github.com/GrahamDumpleton/mod_wsgi/releases)下载文件包，然后解压，编译。假设版本是4.5.3，全部命令如下:
```bash
wget https://github.com/GrahamDumpleton/mod_wsgi/archive/4.5.3.tar.gz  
tar -xvf 4.5.3.tar.gz  
cd mod_wsgi-4.5.3  
./configure  
make  
sudo make install  
```
如果没有报错，那么mod_wsgi就编译好了!
**编译好后，会在apache的模块目录`/usr/lib/apache2/modules/`生成mod_wsgi.so文件。**
###4.Apache配置文件目录结构
Apache的配置文件目录是`/etc/apache2`，该目录下的文件结构如下：
```bash
.
|-- apache2.conf
|-- conf-available
|-- conf-enabled
|-- envvars
|-- magic
|-- mods-available
|-- mods-enabled
|-- ports.conf
|-- sites-available
`-- sites-enabled

```
其中`apache2.conf`是主配置文件，里面包括系统的设置，如Timeout的时长、Log的等级和格式等。`ports.conf`文件配置了监听的端口号，以及是否启用SSL。`envvars`和`magic`里面设置了一些环境变量相关的东西，我没怎么看过。  
剩下的6个目录两两一对，`availabel`文件夹里面是所有的配置，而`enabled`目录里面则是启用的配置。而`conf`、`mods`和`sites`可以分别通过命令`a2enconf`、`a2enmod`、`a2ensite`来启用，启用后会在`enabled`目录下生成一个软链接，指向`available`目录下的同名文件。  
在`apache2.conf`这个文件最后，是一些`IncludeOptional` 语句，用来将`conf-enabled`、`mods-enabled`、`sites-enabled`目录下的配置文件包含到主配置文件中。这样的好处是每个配置文件配置一个条目，比较清晰明了,易于查错。    

###5. 启用wsgi模块
我们需要在`mods-available`目录下新建`mod_wsgi`的load文件，具体操作如下:
```bash
cd /etc/apache2/mod-available  
sudo echo " LoadModule wsgi_module /usr/lib/apache2/modules/mod_wsgi.so" >> wsgi.load  
sudo a2enmod wsgi # 启用wgsi配置
sudo service apache2 restart # 重启Apache2服务
```

###6. 托管Django站点
假设Django项目的`wsgi.py`文件的路径是`/home/yunfeng/Dev/git/mysite/mysite/wsgi.py`，我们需要下面几步来完成Apache对Django项目的托管：

####1. 修改Django项目中的`wsgi.py`和`settings.py`文件
修改`wsgi.py`文件，增加如代码中说明的那几行：
```py
"""                                                                                                                                                           
WSGI config for travel_record project.                                                                                                                        
                                                                                                                                                              
It exposes the WSGI callable as a module-level variable named ``application``.                                                                                
                                                                                                                                                              
For more information on this file, see                                                                                                                        
https://docs.djangoproject.com/en/1.9/howto/deployment/wsgi/                                                                                                  
"""                                                                                                                                                           

import os                                                                                                                                                     

## 增加下面这几行
import sys                                                                                                                                                    
from os.path import dirname, abspath                                                                                                                    
from django.core.wsgi import get_wsgi_application                                                                                                             
PROJECT_DIR = dirname(dirname(abspath(__file__)))                                                                                                             
sys.path.insert(0, PROJECT_DIR)                                                                                                                               
                                                                                                                                                              
#os.environ.setdefault("DJANGO_SETTINGS_MODULE", "travel_record.settings")                                                                                    
os.environ["DJANGO_SETTINGS_MODULE"] = "travel_record.settings"                                                                                               
## 增加结束
                                                                                                                                                              
application = get_wsgi_application()       
```
增加的这几行代码做了2件事：1.将Django项目的的路径加入到系统路径中，使得Apache服务器可以找到`wsgi.py`文件；2. 修改`os.environ`的值，使得多个Django项目同时被Apache托管的时候不会出现串扰的问题。    
接下来修改`settings.py`文件，主要修改的地方有3个：
 1. 将`DEBUG=True`改为`DEBUG=False`
 2. 将`ALLOWEND_HOSTS`里面写上服务器的访问域名或IP地址
 3. 将`TEMPALTES`中的`APP_DIRS`该写成指向模板目录的绝对路径
Django项目里面需要修改的就这2个文件，下面的内容都是在`/etc/apache2`目录下进行操作。  

####2. 在/etc/apache2/sites-available目录下增加网站的配置文件
参照该目录下的`000-default.conf`和Django的教程，写出配置文件mysite.conf如下：
```bash
  <VirtualHost *:8000>                                                                                                                                         
    ErrorLog ${APACHE_LOG_DIR}/error.log                                                                                                                      
    CustomLog ${APACHE_LOG_DIR}/access.log combined                                                                                                           
                                                                                                                                                              
    WSGIScriptAlias / /home/yunfeng/Dev/git/mysite/mysite/wsgi.py                                                                               

    Alias /static/ /home/yunfeng/Dev/git/mysite/mysite/static/                                                                                  
    Alias /media/ /home/yunfeng/Dev/git/mysite/mysite/media/                                                                                  

    <Directory /home/yunfeng/Dev/git/mysite/mysite>                                                                                             
        <Files wsgi.py>                                                                                                                                       
    		Order deny,allow  
    		Allow from all  
        </Files>                                                                                                                                              
    </Directory>                                                                                                                                              

    <Directory /home/yunfeng/Dev/git/mysite/mysite/static/>                                                                                     
    	Order deny,allow  
    	Allow from all  
    </Directory>                                                                                                                                              
	
    <Directory /home/yunfeng/Dev/git/mysite/mysite/static/>                                                                                     
    	Order deny,allow  
    	Allow from all  
    </Directory>                                                                                                                                              
</VirtualHost>       
```
整个配置文件是包含在`VirtualHost`的尖括号里面的一些设置，尖括号开始的地方，`*:8000`表示你希望的项目监听的端口号。  
`ErrorLog`和`CustomLog`设置错误日志和访问日志的路径和格式。  
`WSGIScriptAlias`设置wsgi文件的路径，`Alias`语句托管网站的`static`和`media`目录。  
然后是`<Directory>`标签，用来设置文件和目录的访问权限。**注意对于版本小于2.4的Apache，需要将`<Directory>`标签中的`Order deny,allow`和`Allow from all`改为`Require all granted`。**   
修改完后，执行下面的命令启用这个网站:
```bash
sudo a2ensite mysite.conf
```

####3. 修改/etc/apache2目录下的ports.conf文件
增加针对新建站点的端口号的监听：
```bash
Listen 80
#增加下面这条语句
Listen 8000
```

执行完这3个步骤后，就可以重启Apache服务器，访问站点了：
```bash
sudo service apache2 restart
```
访问站点，如果出现错误的话，可以在Django项目的`settings.py`中启用DEBUG模式，查看输出，进行相应的修改。

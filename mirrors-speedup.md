title: 利用国内开源镜像加速你的包安装
date: 2018-04-18 15:53:35
tags:
 - Linux
 - Python
 - Pip
 - Docker
 - NPM
---
 
由于许多包的存放服务器在国外，国内安装比较慢，因此本文总结了常见的包（例如Python包，Linux不同发行版的包）在国内的开源镜像，加速你的下载，提高安装体验。下面总结了PyPi，Anacoda，NPM， Docker，RubyGems和Linux的国内镜像，并且在GitHub上放置了本文提到的所有的包的配置文件，直接下载使用，具体使用说明访问[这里](https://github.com/vra/mirrors-china)。
<!--more-->
## PyPi 加速
临时加速可以用下面的命令：
```bash
pip install -i https://path/to/pypi/mirror package
```
永久使用的话，需要修改配置文件。对于系统级别的修改，增加下面的配置文件到`/etc/pip.conf`，如果只是自己使用，修改`~/.pip/pip.conf`。
```bash
# file path: /etc/pip.conf or ~/.pip/pip.conf

# ustc, doc: https://lug.ustc.edu.cn/wiki/mirrors/help/pypi
[global]
    index-url = https://mirrors.ustc.edu.cn/pypi/web/simple
    trusted-host=mirrors.ustc.edu.cn


# thu, doc: https://mirrors.tuna.tsinghua.edu.cn/help/pypi/
#[global]
#    index-url = https://pypi.tuna.tsinghua.edu.cn/simple
#    trusted-host=pypi.tuna.tsinghua.edu.cn


# aliyun, doc: http://www.atjiang.com/aliyun-pip-mirror/
#[global]    
#    index-url=http://mirrors.aliyun.com/pypi/simple
#    trusted-host=mirrors.aliyun.com


# 163, no doc.
#[global]
#    index-url=https://mirrors.163.com/pypi/simple
#    trusted-host=mirrors.163.com
```

本文中默认用的中科大的源实际使用的时候，选择自己访问最快的**一个**镜像就可以了，将别的镜像设置注释掉或者删掉。


## Anaconda 包加速
Anaconda是一个Python的包管理系统，包含科学计算常用的包。通过在命令行执行下面的文件就可以使用中科大或者清华的Anaconda镜像了，注意只执行自己访问最快的镜像对应的命令。
```bash
# run this script in terminal
# ustc, doc: https://mirrors.ustc.edu.cn/help/anaconda.html
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes

# thu, doc: https://mirror.tuna.tsinghua.edu.cn/help/anaconda/
#conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
#conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
#conda config --set show_channel_urls yes
```

## NPM 包加速
NPM 是NodeJs的包管理系统，NodeJs的包通过该命令来安装。临时使用镜像来安装某个包可以用下面的命令：
```bash
$ npm --registry http://path/to/npm/mirror install package
```
永久使用某个镜像需要修改`~/.npmrc`，加入下面的某一个镜像即可：
```bash
# file path: ~/.npmrc
# ustc, doc: https://lug.ustc.edu.cn/wiki/mirrors/help/npm
registry=http://npmreg.mirrors.ustc.edu.cn/

# taobao, doc: https://npm.taobao.org/
#registry=http://registry.npm.taobao.org/
```

## Docker 镜像加速
修改`/etc/docker/daemon.json`，加入下面的内容：
```js
// ustc, doc: https://lug.ustc.edu.cn/wiki/mirrors/help/docker
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}

// docker-cn, doc: https://www.docker-cn.com/registry-mirror 
//{
//  "registry-mirrors": ["https://registry.docker-cn.com"]
//}
```

## RubyGems 加速
```bash
# run this script in terminal
# ustc, doc: https://mirrors.ustc.edu.cn/help/rubygems.html
gem sources  #列出默认源
gem sources --remove https://rubygems.org/  #移除默认源
gem sources -a https://mirrors.ustc.edu.cn/rubygems/  #添加科大源

# thu, doc: https://mirror.tuna.tsinghua.edu.cn/help/rubygems/
#gem sources --add https://mirrors.tuna.tsinghua.edu.cn/rubygems/ --remove #https://rubygems.org/
#gem sources -l
```

## Linux 包加速
由于Linux发行版众多，配置各不相同，因此请参考下面的源下面的文档进行对应发行版的配置：
 1. 中科大: <http://mirrors.ustc.edu.cn/help>
 2. 清华：<https://mirrors.tuna.tsinghua.edu.cn/help/AOSP>
 3. aliyun: <https://opsx.alibaba.com/mirror>
 4. 163: <https://mirrors.163.com>

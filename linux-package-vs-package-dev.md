---
title: Linux中的包名"xxx"和"xxx-dev"有什么区别?
date: 2022-07-23 17:40:03
tags:
- Linux
- Debian
- Ubuntu
- CentOS
---
## 1. 引入
在安装包的时候，有时候需要安装`xxx`的包，有时候又需要安装`xxx-dev`的包 (在CentOS系列发行版上则是`xxx-devel`)。这两类包之间又什么区别呢？
<!--more-->

## 2. 结论
不包含`-dev`的包里面包含的是运行所需要的二进制文件或者连接库文件（如`xxx.so`），而包含`-dev`的包则包含包的源码文件（如`.h`文件），为的是在编译使用了这些库的程序的时候，能找到对应的头文件，否则只有二进制文件或者`.so`文件，编译时会报代码找不到头文件的错误。

下面举个例子进行说明。

我们只使用Python的话，用`sudo apt install python`即可，安装后就可以正常使用Python。


如果想要编译一个叫[lxml](https://github.com/lxml/lxml)的库，它依赖Python的源码，例如[这里](https://github.com/lxml/lxml/blob/06631bb0677250cb632638a2c89f4d336360965b/src/lxml/includes/etree_defs.h#L5)的代码依赖`Python.h`这个文件，因此我们需要安装`python-dev`包，把`Python.h`安装到本地上，这样lxml包才能正常安装。


## 3. 参考
1. <https://stackoverflow.com/questions/2358801/what-are-devel-packages>

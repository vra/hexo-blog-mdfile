---
title: Mac 下 Unable to load OpenGL library 的解决办法
date: 2021-11-04 20:03:02
tags:
- macOS
- OpenGL
- Python
- 总结
- 错误汇总
---
## 问题描述
在Mac上使用Pyrender时，出现了OpenGL无法加载的错误，具体复现情况如下:
打开Python的REPL, 输入下面的命令(前提是安装pyrender):
```python
import pyrender
```
报下面的错：
```bash
raise ImportError("Unable to load OpenGL library", *err.args)
ImportError: ('Unable to load OpenGL library', "dlopen(OpenGL, 0x000A): tried: 'OpenGL' (no such file), '/usr/local/lib/OpenGL' (no such file), '/usr/lib/OpenGL' (no such file), '/usr/local/lib/OpenGL' (no such file), '/usr/lib/OpenGL' (no such file)", 'OpenGL', None)
```
这里记录一下解决的办法。
<!--more-->
## 解决办法
解决办法比较简单，首先找到`OpenGL`的安装目录:
```bash
 python3 -c "import OpenGL; print(OpenGL.__path__)"
 # 输出路径
['/usr/local/lib/python3.7/site-packages/OpenGL']
```
有了包路径后，修改包目录下的`platform/ctypesloader.py`文件：
```bash
vi /usr/local/lib/python3.7/site-packages/OpenGL/platform/ctypesloader.py
```
将第35行注释掉，添加新的一行代码：
```python
# 原先的代码
#fullName = util.find_library( name )
# 新的代码
fullName = '/System/Library/Frameworks/OpenGL.framework/OpenGL'
```
然后就可以正常运行了。  
注意：不用确认路径`/System/Library/Frameworks/OpenGL.framework/OpenGL`是否存在，只需原样修改代码即可.


## 参考
1. <https://stackoverflow.com/questions/63475461/unable-to-import-opengl-gl-in-python-on-macos/64021312#64021312>


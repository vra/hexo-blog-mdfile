title: matplotlib的backend浅析
date: 2017-06-13 10:45:53
tags:
 - Python
 - Linux
 - matplotlib
---
在服务器使用`matplotlib`的时候，可能是因为没有装图形化和显示相关的包的原因，总是会出现`backend`相关的错误。所以我调查了下matplotlib中的backend的含义，以及如何处理相关的错误。
![matplotlib](http://matplotlib.org/1.3.0/_static/logo2.png)
<!--more-->


### matplotlib中的backend
matplotlib中，frontend就是我们写的python代码，而backend就是负责显示我们代码所写图形的底层代码。因为不同使用环境下硬件情况不同，所以后端是跟具体的硬件和显示条件相关的。


### backend的类别
backend又分为两类，一类是`interface backend`，又叫做`interactive backend`，这一类是表示跟显示到屏幕相关的后端；另一类是`hardcopy backend`，又叫做`non-interactive backend`，这一类是写入到文件相关的后端。下面两图分别是non-interactive backend和interactive backend的具体值：
![non-interactive backend](http://7xlt5t.com1.z0.glb.clouddn.com/non-iteractive-backend.png)


![interactive backend](http://7xlt5t.com1.z0.glb.clouddn.com/iteractive-backend.png)
在python中，可以通过如下的命令来获取当前机器支持的这两种后端：
```python
import matplotlib
matplotlib.rcsetup.interactive_bk # 获取 interactive backend
matplotlib.rcsetup.non_interactive_bk # 获取 non-interactive backend
matplotlib.rcsetup.all_backends # 获取 所有 backend
```
在我们实验室的GPU服务器上，得到的结果如下：
{% asciinema avw1ng2fbxp5fdk8b2cncpbp0 %}


### 设置backend
有4种方式可以来设置matplotlib的backend，而且下列越后面的设置方式，优先级越高，也就是后面的设置会覆盖前面的设置。  
####1. 通过设置`matplotlibrc`的配置文件来设置
注意`matplotlibrc`文件不一定在你的家目录下，可以通过如下命令来获取其存放位置:
```python
import matplotlib
matplotlib.get_configdir()
u'/home/yunfeng/.config/matplotlib'
```
得到配置文件路径后，打开这个文件，写入如下一行来设置backend:
```bash
backend : WXAgg   # use wxpython with antigrain (agg) rendering
```
其中`WXAgg`可以换成任意的你的系统支持的backend类型。  
**注意：在backend的名字中是不区分大小写的，所以`Qt4Agg`和`qt4agg`是等价的。**

####2. 通过`MPLBACKEND`环境变量来设置backend
下面两种方式都可以:
```bash
## 方式1. 先export MPLBACKEND在执行python文件
$ export MPLBACKEN='Agg'
$ python works.py

## 方式2. 在python命令前加MPLBACKEND='XXX'
$ MPLBACKEND='Agg' python works.py
```

####3. 通过`-d`选项来设置
使用方法如下：
```bash
$ python script.py -dbackend
```
因为这种方式很容易和脚本内部的参数解析冲突，所以不建议使用这种方式，而是通过`MPLBACKEND`参数的方式2来设置。

####4. 通过`matplotlib.use()`函数来设置
使用方式如下：
```python
import matplotlib as mpl
mpl.use('Agg')
```
**再次提醒下，注意这4种方式的优先级：4>3>2>1，后面的设置会覆盖前面的设置。**
### 解决问题
####1. GPU集群执行`import matplotlib.pyplot as plt`的错误
错误信息可能如下：
```bash
** (test_net_multi.py:23890): WARNING **: Could not open X display

(test_net_multi.py:23890): Gdk-CRITICAL **: gdk_cursor_new_for_display: assertion 'GDK_IS_DISPLAY (display)' failed
Traceback (most recent call last):
  File "./tools/test_net_multi.py", line 13, in <module>
    from fast_rcnn.test_multi import test_net
  File "/data2/yunfeng/Lab/RstarCNN/tools/../lib/fast_rcnn/__init__.py", line 9, in <module>
    from . import train
  File "/data2/yunfeng/Lab/RstarCNN/tools/../lib/fast_rcnn/train.py", line 15, in <module>
    import caffe
  File "/data2/yunfeng/Lab/RstarCNN/tools/../caffe-fast-rcnn/python/caffe/__init__.py", line 1, in <module>
    from .pycaffe import Net, SGDSolver
  File "/data2/yunfeng/Lab/RstarCNN/tools/../caffe-fast-rcnn/python/caffe/pycaffe.py", line 14, in <module>
    import caffe.io
  File "/data2/yunfeng/Lab/RstarCNN/tools/../caffe-fast-rcnn/python/caffe/io.py", line 2, in <module>
    import skimage.io
  File "/usr/lib64/python2.7/site-packages/skimage/io/__init__.py", line 15, in <module>
    reset_plugins()
  File "/usr/lib64/python2.7/site-packages/skimage/io/manage_plugins.py", line 93, in reset_plugins
    _load_preferred_plugins()
  File "/usr/lib64/python2.7/site-packages/skimage/io/manage_plugins.py", line 73, in _load_preferred_plugins
    _set_plugin(p_type, preferred_plugins['all'])
  File "/usr/lib64/python2.7/site-packages/skimage/io/manage_plugins.py", line 85, in _set_plugin
    use_plugin(plugin, kind=plugin_type)
  File "/usr/lib64/python2.7/site-packages/skimage/io/manage_plugins.py", line 255, in use_plugin
    _load(name)
  File "/usr/lib64/python2.7/site-packages/skimage/io/manage_plugins.py", line 299, in _load
    fromlist=[modname])
  File "/usr/lib64/python2.7/site-packages/skimage/io/_plugins/matplotlib_plugin.py", line 3, in <module>
    import matplotlib.pyplot as plt
  File "/usr/lib64/python2.7/site-packages/matplotlib/pyplot.py", line 114, in <module>
    _backend_mod, new_figure_manager, draw_if_interactive, _show = pylab_setup()
  File "/usr/lib64/python2.7/site-packages/matplotlib/backends/__init__.py", line 32, in pylab_setup
    globals(),locals(),[backend_name],0)
  File "/usr/lib64/python2.7/site-packages/matplotlib/backends/backend_gtk3agg.py", line 11, in <module>
    from . import backend_gtk3
  File "/usr/lib64/python2.7/site-packages/matplotlib/backends/backend_gtk3.py", line 58, in <module>
    cursors.MOVE          : Gdk.Cursor.new(Gdk.CursorType.FLEUR),
TypeError: constructor returned NULL
```
这是因为服务器没有装显示相关的包，可以通过在上述第2种方式来设置`MPLBACKEN='Agg'`即可解决这个问题，因为Agg是non-interactive backend，所以不会要求显示图片，所以也不会再报错了。举个例子，如果你的TorqueServer的配置文件如下：
```bash
#PBS    -N  v_test_60k
#PBS    -o  /home/yunfeng/logs/v_test_60k.out
#PBS    -e  /home/yunfeng/logs/v_test_60k.err
#PBS    -l nodes=1:gpus=2:D
#PBS    -r y
cd $PBS_O_WORKDIR
echo Time is `date`
echo Directory is $PWD
echo This job runs on following nodes:
cat $PBS_NODEFILE
cd /data10/yunfeng/Dev/tsn
python v_test_60k.py 2>&1 |tee ~/logs/v_test_60k.log
```
在python执行那行行首，增加`MPLBACKEND=Agg`，即改为如下内容：
```bash
#PBS    -N  v_test_60k
#PBS    -o  /home/yunfeng/logs/v_test_60k.out
#PBS    -e  /home/yunfeng/logs/v_test_60k.err
#PBS    -l nodes=1:gpus=2:D
#PBS    -r y
cd $PBS_O_WORKDIR
echo Time is `date`
echo Directory is $PWD
echo This job runs on following nodes:
cat $PBS_NODEFILE
cd /data10/yunfeng/Dev/tsn
MPLBACKEND=Agg python v_test_60k.py 2>&1 |tee ~/logs/v_test_60k.log
```

####2. GPU服务器上使用matplotlib显示图片
由于服务器没有安装图形化显示界面，所以使用默认的matplotlib设置会有一些问题，图片没法正常显示。解决方法是在python文件中增加如下两行：
```python
import matplotlib as mpl
mpl.use('Qt4Agg')
```
在Jupyter notebook和VNC连过去后，这种设置都可以正确地显示图片。  
**注意，这两行必须在`import matplotlib.pyplot as plt`之前插入，否则在plt引入后，上面的设置就没有效果了。**
至于为什么是`Qt4Agg`，我是一个个后端一一试出来的，应该跟服务器安装的显示包有关系，但是我暂时还没弄懂该如何查看。

### 参考
1. <http://matplotlib.org/1.3.0/faq/usage_faq.html>

title: 关闭Caffe和TensorFlow运行时的日志输出
date: 2017-12-11 11:38:45
tags:
 - Caffe
 - Tensorflow
---

简言之2条命令即可：
```bash
# 在命令行下
# Caffe
$ GLOG_minloglevel=2 caffe-command
# Tensorflow
$ TF_CPP_MIN_LOG_LEVEL=3 tensorflow-command
```
或者在python文件中，**import caffe或tensorflow之前**，执行如下的语句：
```python
# 在Python文件中
# Caffe
import os
os.envrion['GLOG_minloglevel'] = '2'

# Tensorflow
import os
os.envrion['TF_CPP_MIN_LOG_LEVEL'] = '3'
```

参考：

0. <https://stackoverflow.com/questions/29788075/setting-glog-minloglevel-1-to-prevent-output-in-shell-from-caffe>
1. <http://littlewhite.us/archives/157>
2. <https://stackoverflow.com/questions/38073432/how-to-suppress-verbose-tensorflow-logging>

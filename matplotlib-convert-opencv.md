---
title: matplotlibt图像转OpenCV图像
date: 2022-06-24 23:14:44
tags:
- OpenCV
- Matplotlib
- Python
---
## 1. 概述
有时候，我们需要使用Matplotlib库强大的绘图函数来在numpy.ndarray格式的图像上进行一些可视化，比如关键点绘制，投影点绘制。绘制完后，还需要把matplotlib的figure对象转换为numpy.ndarray 格式的对象，方便和原图进行比较。有时候为了可视化的美观，需要验证保证转换后的图像与原始图像大小一致。这里记录一下操作的流程，以及一些常遇到的问题。
<!--more-->

## 2. 原理
核心原理是利用matplotlib.pyplot的`imshow`函数来显示np.ndarray格式的图像，然后进行可视化绘制，再通过matplotlib.pyplot.figure.canvas的`tostring_rgb`函数来将图像转换为string，在用numpy的`fromstring`函数将string转换为np.ndarray，即为我们所求。

示例代码如下：
```python
import cv2
import matplotlib.pyplot as plt
import numpy as np

# 读取 numpy.ndarray格式的图像
img = cv2.imread('/path/to/my.jpg')
h, w = img.shape[:2]

# 创建figure对象
fig = plt.figure()
# 显示图像
plt.imshow(img)

# 添加函数绘制代码
...

# 绘制画布
fig.canvas.draw()

# 转换plt canvas为string，再导入numpy
vis_img = np.fromstring(fig.canvas.tostring_rgb(), dtype=np.uint8)
# 设置numpy数组大小为图像大小
vis_img.shape = (h, w, 3)

plt.close()

cv2.imwrite('/path/to/vis_img.jpg', vis_img)
```

## 3. 几个关键点
上述代码是简单的原理，但要达到保存的`vis_img`对象与`img`对象完全等大小，还需要设置figure对象的size，具体实现是通过`set_size_inches`函数，传入原始图像的宽和高除以dpi的值：
```python
fig = plt.figure()
fig.set_size_inches(w/fig.dpi, h/fig.dpi)
```
注意是宽在前面，高在后面。

还有一个很关键的点是需要去除matplotlib设置的padding白边，否则在相同尺寸的情况下，包含白边显得里面的内容变小了：
```python
plt.gca().set_position((0, 0, 1, 1))
```

为了不显示横纵坐标轴，需要添加`plt.axis('off')`语句。

为了能在无GUI的环境（比如SSH连到的Linux 服务器）这个脚本也能正常工作，需要采用`Agg` 这个backend：
```python
import matplotlib
matplotlib.use('Agg')
```
插句题外话，`Agg`这个backend原来是来自于[Anti-Grain Geometry](http://agg.sourceforge.net/antigrain.com/) 2D渲染库，2002年开始开发，距今已有20年历史了，Respect。

此外由于matploltlib的`imshow`需要RGB格式的图像，而OpenCV图像格式为BGR，需要做转换。

## 4. 完整代码
结合上一部分的几个关键点，最终的代码如下：
```python
import cv2
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
import numpy as np


img = cv2.imread('/path/to/my.jpg')
img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

h, w = img.shape[:2]

fig = plt.figure()
fig.set_size_inches(w/fig.dpi, h/fig.dpi)

plt.imshow(img)

# 开始 matplotlib的绘制
# ...

# 关掉坐标轴的显示
plt.axis('off')

# 加这一句，避免matplotlib的自动padding导致的空白
plt.gca().set_position((0, 0, 1, 1))

fig.canvas.draw()

# 转换plt canvas为string，再导入numpy
vis_img = np.fromstring(fig.canvas.tostring_rgb(), dtype=np.uint8)
# 设置numpy数组大小为图像大小
vis_img.shape = (h, w, 3)
# 将RGB格式转换为BGR格式
vis_img = cv2.cvtColor(vis_img, cv2.COLOR_RGB2BGR)

plt.close()

cv2.imwrite('/path/to/vis_img.jpg', vis_img)
```
需要注意的是，直接执行这段代码虽然可以得到你想要的结果，但本身是没有意义的，最核心的matplotlib调用需要你自己填写。
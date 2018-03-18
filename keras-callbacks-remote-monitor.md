title: Keras Callback之RemoteMonitor
date: 2018-03-18 18:35:41
tags:
 - Deep Learning
 - Keras
 - TensorFlow
 - Python
 - Linux
---
### 概述
Keras提供了一系列的回调函数，用来在训练网络的过程中，查看网络的内部信息，或者控制网络训练的过程。`BaseLogger`、`ProgbarLogger`用来在命令行输出Log信息（默认会调用）， `EarlyStopping`、`ReduceLROnPlateu`分别用来提前终止训练和自动调整学习率，改变网络训练过程；而今天要介绍的`RemoteMonitor`则用来**实时**输出网络训练过程中的结果变化情况，包括训练集准确率(`accu`)、训练集损失值(`loss`)、验证集准确率(`val_acc`)、验证集损失值(`val_loss`)，用户也可以自己修改需要显示的数据。一图胜千言，看看下面的结果图吧：
![](http://7xlt5t.com1.z0.glb.clouddn.com/keras_viz.png)

这个图是在浏览器中打开得到，Keras使用了Flask搭建了一个简单的服务器，然后采用D3.js来可视化数据。下面详细介绍可视化的过程吧
<!--more-->
### 安装依赖

 1. 首先，你得安装Keras: `sudo pip install keras`或者没有管理员权限的话，执行：`pip install --user keras`，下同
 2. 安装Flask网络框架： `sudo pip install Flask`
 3. 安装gevent,gevent是一个并发框架，可以监听网络训练，并将结果传回网络服务，安装命令：`sudo pip install gevent`


### 下载 Hualos
这是Keras作者写的Keras可视化的项目，其中包括了D3.js这些数据可视化库，也有一个简单的Flask app，即api.py文件，来运行我们的网络参数可视化。具体安装过程如下：
```bash
 git clone https://github.com/fchollet/hualos.git
 cd hualos
 python api.py
```
这样网络服务器就建立了，该服务会监听`http://localhost:9000`端口，你打开浏览器访问该网址，会看到一个初始的页面，我们接下来要做的是在训练网络的时候增加回调函数RemoteMonitor，将网络参数显示到该网址的页面上。


### 在Keras训练网络中加入RemoteMonitor回调函数
这一步只需要在keras的代码里面增加3行即可：
```python
 ## 1. import RemoteMonitor
 from keras.callbacks import RemoteMonitor

 ## 2. 创建RemoteMonitor对象
 remote = RemoteMonitor()

 ## 3. 在model.fit中增加回调函数设置
 model.fit(
 	...,
 	...,
 	callbacks=[remote]
 	)
```

我修改了<https://github.com/fchollet/keras/blob/master/examples/mnist_cnn.py>，增加了以上3行，整个文件如下：
```python
 '''Trains a simple convnet on the MNIST dataset.

Gets to 99.25% test accuracy after 12 epochs
(there is still a lot of margin for parameter tuning).
16 seconds per epoch on a GRID K520 GPU.
'''

from __future__ import print_function
import keras
from keras.datasets import mnist
from keras.models import Sequential
from keras.layers import Dense, Dropout, Flatten
from keras.layers import Conv2D, MaxPooling2D
from keras import backend as K
from keras.callbacks import RemoteMonitor

batch_size = 128
num_classes = 10
epochs = 40

# input image dimensions
img_rows, img_cols = 28, 28

# the data, shuffled and split between train and test sets
(x_train, y_train), (x_test, y_test) = mnist.load_data()

if K.image_data_format() == 'channels_first':
    x_train = x_train.reshape(x_train.shape[0], 1, img_rows, img_cols)
    x_test = x_test.reshape(x_test.shape[0], 1, img_rows, img_cols)
    input_shape = (1, img_rows, img_cols)
else:
    x_train = x_train.reshape(x_train.shape[0], img_rows, img_cols, 1)
    x_test = x_test.reshape(x_test.shape[0], img_rows, img_cols, 1)
    input_shape = (img_rows, img_cols, 1)

x_train = x_train.astype('float32')
x_test = x_test.astype('float32')
x_train /= 255
x_test /= 255
print('x_train shape:', x_train.shape)
print(x_train.shape[0], 'train samples')
print(x_test.shape[0], 'test samples')

# convert class vectors to binary class matrices
y_train = keras.utils.to_categorical(y_train, num_classes)
y_test = keras.utils.to_categorical(y_test, num_classes)

model = Sequential()
model.add(Conv2D(32, kernel_size=(3, 3),
                 activation='relu',
                 input_shape=input_shape))
model.add(Conv2D(64, (3, 3), activation='relu'))
model.add(MaxPooling2D(pool_size=(2, 2)))
model.add(Dropout(0.25))
model.add(Flatten())
model.add(Dense(128, activation='relu'))
model.add(Dropout(0.5))
model.add(Dense(num_classes, activation='softmax'))

model.compile(loss=keras.losses.categorical_crossentropy,
              optimizer=keras.optimizers.Adadelta(),
              metrics=['accuracy'])

remote = RemoteMonitor()
model.fit(x_train, y_train,
          batch_size=batch_size,
          epochs=epochs,
          verbose=1,
          validation_data=(x_test, y_test),
          callbacks=[remote]
        )
score = model.evaluate(x_test, y_test, verbose=0)
print('Test loss:', score[0])
print('Test accuracy:', score[1])
```
注意15、64和70行是我新加的。  
修改完文件后，在命令行运行该文件，在浏览器打开`http://localhost:9000`网址，就可以看到你训练时的参数了。注意结果是每个epoch输出一次。


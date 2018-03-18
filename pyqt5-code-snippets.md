title: PyQt5 代码片段集合
date: 2018-01-13 18:34:15
tags:
 - PyQt5
 - Python
 - Qt 

---
PyQt5是Qt的Python绑定库，既有Qt的强大，又有Python语言的简洁，要实现一个实际场景的GUI程序的时候，确实非常实用而且代码量不是太多。这里我总结了最近写一个界面时用到的代码片段，希望以后用到的时候能及时拾起来，也希望能帮助到别人。 此外我将这个内容也放到[GitHub](https://github.com/vra/pyqt5-code-snippets/blob/master/pyqt_code_snippets.md)上，有兴趣的同学可以收藏下。
<!--more-->

### 安装
目前PyQt主要是4和5版本，因为两者不兼容，因此官方建议使用PyQt5, 这里以Python3 为例进行说明。PyQt5通过pip3来安装，同时别忘了需要安装SIP，这是将Python代码转换为C或C++代码的工具。
```bash
pip3 install PyQt5 SIP
```
安装好后可以使用下面这个代码片段测试安装是否成功，如果可以正常运行说明安装已经成功：
```python
import sys
from PyQt5 import QtCore, QtWidgets
from PyQt5.QtWidgets import QMainWindow, QLabel, QGridLayout, QWidget
from PyQt5.QtCore import QSize    
     
class HelloWindow(QMainWindow):
    def __init__(self):
        QMainWindow.__init__(self)
 
        self.setMinimumSize(QSize(640, 480))    
        self.setWindowTitle("Hello world") 
        
        centralWidget = QWidget(self)          
        self.setCentralWidget(centralWidget)   
 
        gridLayout = QGridLayout(self)     
        centralWidget.setLayout(gridLayout)  
 
        title = QLabel("Hello World from PyQt", self) 
        title.setAlignment(QtCore.Qt.AlignCenter) 
        gridLayout.addWidget(title, 0, 0)
 
 
if __name__ == "__main__":
    app = QtWidgets.QApplication(sys.argv)
    mainWin = HelloWindow()
    mainWin.show()
    sys.exit( app.exec_() )
```

### 整体框架
```python
from PyQt5.QtCore import QDir, Qt, QUrl, QObject
from PyQt5.QtMultimedia import QMediaContent, QMediaPlayer
from PyQt5.QtMultimediaWidgets import QVideoWidget
from PyQt5.QtWidgets import (QApplication, QFileDialog, QHBoxLayout, QLabel,
        QPushButton, QSizePolicy, QSlider, QStyle, QVBoxLayout, QWidget,
        QGridLayout, QFileDialog,QMainWindow,QWidget, QPushButton, QAction,
        QSplitter, QFrame)
from PyQt5.QtGui import QIcon, QFont

class Demo(QMainWindow):
    def __init__(self, numCam=6, parent=None):
        super(ReIDDemo, self).__init__(parent)
        self.setWindowTitle("Video Person Re-ID Demo")
        ...

if __name__ == '__main__':
    app = QApplication(sys.argv)
    #app.setFont(QFont("Consolas", 10))
    demo = Demo(numCam=6)
    #demo.resize(1200, 800)
    demo.setGeometry(400, 30, 1200, 1000)
    demo.show()
    sys.exit(app.exec_())
```

下面就是各个组件的使用方式，只列出了一些常用的功能，别的功能还得在使用的时候再查找。  


### Button
```python
btn = QPushButton()
btnsetEnabled(False)
btn.SetText("Open")
btn.setStyleSheet('{background-color: #A3C1DA; color: red;}')
lbl.setFont(QFont("Consolas", 12))
btn.clicked.connect(func)

def func():
    pass
```

### Label
```python
lbl = QLabel()
lbl.setText('Information')
lbl.setText("<font color='red'>Information</font>")
lbl.setFont(QFont("Consolas", 12))
```

### Slider
```python
positonSlider = QSlider(Qt.Horizontal)
positonSlider.setRange(0, 0)
positonSlider.sliderMoved.connect(setPosition)

def setPosition(position):
    pass
```

### MediaPlayer
```python
player = QMediaPlayer(None, QMediaPlayer.VideoSurface)
VideoWidget = QVideoWidget()
player.setVideoOutput(VideoWidget)
player.stateChanged.connect(playerStateChanged)
player.positionChanged.connect(playerPositionChanged)
player.durationChanged.connect(playerDurationChanged)
player.setMedia(QMediaContent(QUrl.fromLocalFile("/home/user/a.mp4"])))
player.play()   # 开始播放
player.pause()  # 暂停播放

def playerStateChanged(state):
    pass
def playerPositionChanged(position):
    pass
def playerDurationChanged(duration):
    pass
```

### FileDialog
```python
names = QFileDialog.getOpenFileName(self, "Open Query Video", 'd:/3rd')
if names[0]:
    pass
else:
    pass
```

### Frame
```python
leftWidget = QFrame()
leftWidget.setFrameShape(QFrame.StyledPanel)
leftWidget.setLayout(left)
```

### QHBoxLayout
```python
controlLayout = QHBoxLayout()
controlLayout.setContentsMargins(0, 0, 0, 0)
controlLayout.addWidget(btn)
controlLayout.addWidget(slider)
```

### QVBoxLayout
```python
controlLayout = QVBoxLayout()
controlLayout.setContentsMargins(0, 0, 0, 0)
controlLayout.addWidget(btn)
controlLayout.addWidget(slider)
```

### QGridLayout
```python
left = QGridLayout()
left.setSpacing(10)
left.addWidget(lblQuery, 0, 0, 1, 2)
left.addLayout(queryVideo.layout, 1, 0, 1, 2)
left.addWidget(btnOpen, 2, 0)
left.addWidget(btnSearch, 2, 1)
left.addWidget(lblHold, 5, 0, 1, 2)
left.setRowStretch(0, 1)
left.setRowStretch(1, 4)
left.setColumnStretch(1, 4)
```

### Splitter
```python
leftWidget = QFrame()
rightWidget = QFrame()
spliter1 = QSplitter(Qt.Horizontal)
spliter1.addWidget(leftWidget)
spliter1.addWidget(rightWidget)
```

### 教程和资源
 1. [PyQt5 中文教程](https://www.gitbook.com/book/maicss/pyqt5), 上手非常好的教程
 2. [PyQt5 实例教程](https://pythonprogramminglanguage.com/pyqt/), 实例很全面

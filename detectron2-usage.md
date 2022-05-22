---
title: detectron2 使用总结
date: 2020-03-14 23:21:10
tags:
 - Computer Vision
 - Detection
 - Segmentation
 - Deep Learning
 - Detectron2
 - Pytorch
 - Python
 - Linux
 - 总结
---
### 概述
1. detectron2 大部分代码都需要GPU
2. detectron2 主要是用于检测和分割的代码框架，像分类这种任务的代码暂时没有
3. 官方示例有一些是基于Colab的，需要科学上网才能访问
### 安装依赖
```bash
sudo pip install -U torch==1.4+cu100 torchvision==0.5+cu100 -f https://download.pytorch.org/whl/torch_stable.html
sudo pip install cython pyyaml==5.1 --ingnore-installed

# 安装 cocoapi
sudo pip install -U 'git+https://github.com/cocodataset/cocoapi.git#subdirectory=PythonAPI'
```
其中cocoapi 需要从GitHub下载代码，如果安装太慢，可以先clone下代码，再进`PythonAPI`子目录，运行`setup.py`安装:
```bash
git clone https://github.com/cocodataset/cocoapi.git
cd cocoapi/PythonAPI
sudo python3 setup.py install
```

### 安装 detectron2
这里直接安装编译好的二进制文件。
```bash
sudo pip install detectron2 -f https://dl.fbaipublicfiles.com/detectron2/wheels/cu100/index.html
```
如果文件下载太慢或者超时，可以手动在浏览器里面下载好文件，再用下面的命令安装（假设下载的`whl`文件是`xxx.whl`):
```bash
sudo pip install xxx.whl
```
安装完后，打开 Python 命令行，执行下面的命令，如果不报错，说明安装成功：
```python
import detectron2
```

### 测试
为了测试，需要下载 detectron2 的源代码，基于 `demo/demo.py` 进行简单的测试：
```bash
git clone https://github.com/facebookresearch/detectron2
python3  detectron2/demo/demo.py --config-file detectron2/configs/COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml --input ~/test.jpg --opts MODEL.WEIGHTS  detectron2://COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x/137849600/model_final_f10217.pkl
```
**注意上述代码需要在 detectron2 的 git 仓库外面执行，否则会报错。**
测试时输入支持单张图片、多张图片、单个图片文件夹、网络摄像头以及视频文件，每种情况参数设置如下：
```bash
# 单张图片
--input test.jpg
# 多张图片
--input test1.jpg test2.jpg test3.jpg
# 单个图片文件夹
--input imgs/*.jpg
# 网络摄像头
--webcame
# 视频文件
--video-input test.mp4
```

``--opts MODEL.WEIGHTS` 表示测试用的模型参数，可以是一个本地目录，也可以是一个 `detectron2://`开头的一个模型路径，这时会先下载模型到本地再测试:
```bash
# 使用本地的模型参数
--opts MODEL.WEIGHTS ./model_final_f10217.pkl
# 使用网络模型地址
--opts MODEL.WEIGHTS ./model_final_f10217.pkl
```
模型的名字可以在 [Model Zoo](https://github.com/facebookresearch/detectron2/blob/master/MODEL_ZOO.md) 查看。

### 训练
训练代码参考 `tools/train_net.py`，目前Detection看。

### 一些代码分析
1. DefaultTrainer 是针对目前常用的Detection设置而写的一个类，为了不用修改太多就直接复现最佳结果。但另一方面，由于有比较多的假设情况，因此通用性有所降低
2. SimpleTrainer 是 DefaultTrainer 的父类，限制条件更少，对于做新的研究任务，作者推荐继承 SimpleTrainer 来修改
3. 代码支持多机多卡多进程，基于 Pytorch 的多级多卡代码写了一些wrapper
4. 代码注释很完善，而且其中很多是给用户怎么基于现在代码进行修改来跑新的网络、做新的任务，有些地方说的很细致，这一点很棒


### 一些资源
1. [文档](https://detectron2.readthedocs.org/)
2. [Git 仓库](https://github.com/facebookresearch/detectron2)
---
title: VitPose 论文阅读
date: 2022-06-11 20:12:02
tags:
- 论文阅读
- Computer Vision
- Paper
- Pose Estimation
- ViT
- Transformer
---

## 1. 概述
VitPose是最近出来的一篇用Transformer结构做人体2D关键点估计的论文，采用比较简单的Transformer结构就能在MS COCO 测试集上取得比较好的结果，挺吸引人的。论文不长，这周末读了一遍，感觉值得借鉴的地方挺多，这里我用自己的语言描述论文的细节，同时把自己的一些疑惑和思考写下来，欢迎讨论交流。

论文标题: ViTPose: Simple Vision Transformer Baselines for Human Pose Estimation
论文地址：<https://arxiv.org/abs/2204.12484>
代码地址：<https://github.com/ViTAE-Transformer/ViTPose>

注：本文中框图和表格均来自原论文。

<!--more-->
## 2. 摘要和引入
Vison Transformer 在视觉识别任务中效果优秀，在识别但还没有人在姿态估计任务上验证这种结构的有效性。这篇论文提出了名为VitPose的用于姿态估计的Transformer网络，使用普通ViT结构作为Backbone，结合一个轻量级的Decoder，就能在MS COCO 关键点估计bechmark上达到SOTA。


## 3. 继续阅读前的几个疑问
读完摘要和Introduction部分，我决定继续精读这篇论文，因此在进一步阅读前，为了提升对论文的理解程度，我想出了下面的问题，希望在读完剩余部分的时候，这些问题都能得到回答:

1. 如何确定SOTA结果中MAE和Transformer网络结构的贡献?
2. 100M到1B参数的变化是通过哪个模块的变化调节的?
3. 是基于Heatmap还是Regression的思路?
4. 只针对单人场景还是多人场景也OK?
5. 速度如何？

带着这些疑问，咱们继续往下看。

## 4. 实现细节
### 4.1 整体结构
网络结构设计比较简单，整体为采用ViT backbone + decoder的形式。

![vitpose framework](/imgs/vitpose/vitpose_framework.jpg)
backbone分为patch embedding和多个transfomer模块。patch embedding将图像分为dxd的patch块。

而每个transfomer层包含 multi-head self-attention(MHSA) 与 feed-forward network (FFN) 模块。多个transfomer层堆叠，构成了backbone。

backbone根据计算量大小，选用了Vit-B, ViT-L，ViT-H[3]以及ViTAE-G[4]。

#### 4.1.1 decoder 选择
由于backbone采用ViT现有的结构，因此在decoder的选取上，作者选择了两种结构进行了对比:
1. 经典Decoder结构，两个Deconv（+BN+ReLU) + 1个1x1 conv，每个deconv上采样2倍，最终输出feature map大小为输入的1/4倍
![](/imgs/vitpose/vitpose_classic_decoder.jpg)
2. 双线性差值上采样4倍，然后是ReLU+3x3conv，不过论文中公式与描述不符，ReLU在双线性上采样之前，需要看代码实现具体是哪一种。
![](/imgs/vitpose/vitpose_simple_decoder.jpg)

方案1非线性更高，因此在CNN的结构中使用比较多。而这篇论文也验证了由于Transformer强大的学习能力，即使像方案2这样的的简单decoder，也能达到很高的精度：

![](/imgs/vitpose/vitpose_t1.png)
可以看到，ResNet系列在方案1上的结果远高于方案2，说明CNN结构的学习能力需要强有力的decoder来进一步加强，而VitPose结构则不需要，这需要归功于ViT结构的强大学习能力

如果光讲结构确实比较单一，所以论文也在好几个方面验证了ViTPose的优良特性。

### 4.2 灵活性
#### 4.2.1 预训练上的灵活性
一般情况下backbone都需要ImageNet上预训练。这篇论文提出了三种预训练方案：
1. 采用ImagNet预训练分类任务，比较经典的方法，数据集总共1M图片
2. 采用MS COCO 预训练MAE任务，将75%的patch随机的mask掉，然后让网络学习恢复这些patch，数据集共150K图片
3. 任务框架同方案2，不过数据集采用MS COCO + AI Challenger，共500K图片

具体实现是将MS COCO和AI Challenger 中的单个人体crop出来，与ImageNet单个object的数据分布保持一致。然后在3个数据集上分别训练1600个epoch，再在MS COCO 上fine tune 210个epoch。

这个训练周期确实有点出乎意料地长……

采用VitPose-B结构，在MS COCO val set上，三种预训练方案的结果如下:
![](/imgs/vitpose/vitpose_t2.jpg)
可以看到使用MS COCO + AI Challenger，在只有一半数据量的情况下，可以达到比ImageNet更好的效果。

#### 4.2.2 分辨率上的灵活性
ViTPose可以通过使用更大的输出尺寸来训练，也可以通过减小backbone中的下采样来构造更大尺度的feature map，这两种操作都能提高精度，具体如下：
更大尺寸的输入：直接缩放原始图像，得到对应大小的输入
更大尺寸的特征：降低采样倍数，修改patch层的stride参数，

另外提一下，这个特性应该是CNN和ViT结构都通用的。

结果如下：
![](/imgs/vitpose/vitpose_t3.jpg)
可以看到分辨率越大结果越高

#### 4.2.3 Attention种类上的灵活性
众所周知，Transformer中的Attention的计算量是Feature map 尺寸的平方，因此是很大的，而且显存占用也很大。因此作者用了Shift Window 和 Pooling Window 两种方案来缓解这个问题，结果如下：

![](/imgs/vitpose/vitpose_t4.jpg)
单纯的网络显存占用太多，因此不得不采用fp16才能训起来……

#### 4.2.4 finetune的灵活性
与NLP任务中一样，作者验证了只固定MHSA模块的参数，精度下降不多，而固定FFN的参数，则精度下降明显，因此作者认为MHSA更偏向**与任务无关**，而FFN则更具体任务关系更密切。

![](/imgs/vitpose/vitpose_t5.jpg)

#### 4.2.5 多任务上的灵活性
作者还尝试了这样一个实验，采用同一个backbone，多个decoder，每个decoder对应一个数据集的任务，实验验证一次训练，多个数据集上的结果都能比较好，且比单个数据集精度有提升:

![](/imgs/vitpose/vitpose_t6.jpg)

### 4.3 蒸馏
这篇论文比较有意思的一个点是提出了一个基于Transformer的蒸馏方法，与常见的用loss来监督Teacher和Student网络的思路不太一样，具体如下:
1. 在大模型的patch embedding后的visual token后面增加一个知识token模块，并进行随机初始化
2. 固定大模型的参数，只训练知识token模块
3. 将训练好的知识token模块接到小模型的visual token后面，且固定知识token的参数，只训练小模型的其他参数

通过这样的流程，将所有的知识都融合到了知识token模块的参数里面，并且从大模型传递到小模型，感觉理解起来也是很直观很有画面感。

结果如下：

![](/imgs/vitpose/vitpose_t7.jpg)


### 4.4 与SOTA对比
 实现细节中作者说明了，采用姿态估计中Top-Down的方案，即先用一个检测器检测出单个人体框，然后对人体框进行姿态估计。本文中方案其实是后面这一步。第一步的检测器在COCO的val集上用的是SimpleBaseline[1]，而在最后的COCO test-dev集上，与SOTA方案的比较实验中，采用了Bigdet[2]。

SOTA结果是在576x432输入，采用1B参数量的ViTAE-G作为backbone，使用MS COCO + AI Challenger训练的情况下获得的，具体如下：
![](/imgs/vitpose/vitpose_t8.jpg)
![](/imgs/vitpose/vitpose_t8.jpg)


## 5 几个疑问的答案：
相信经过上面的细节描述，我们对开头的几个疑问中的一些问题已经有明确的答案了
1. 如何确定SOTA结果中MAE和Transformer网络结构的贡献? -> 
2. 100M到1B参数的变化是通过哪个模块的变化调节的? -> 通过修改backbone的结构来控制参数大小 
3. 是基于Heatmap还是Regression的思路? -> Heatmap
4. 只针对单人场景还是多人场景也OK? -> 只针对单人场景，且需要额外的前置detector
5. 速度如何？ -> 速度应该是比较慢的，训练周期比较长，网络比较大


## 6 思考
1. 采用强大的Transformer结构，之前的很多trick都可以省略，包括skip-connection 等
2. Knowledge Token的思路很新颖挺有意思的，感觉可以用在所有的Transformer蒸馏里面
3. 虽然论文强调只用了一个普通的ViT结构来做姿态估计，但是为了达到较高的精度，后面还是挺多提点的实验


## 7 参考
[1] SimpleBaseline: https://arxiv.org/abs/1804.06208
[2] Bigdetection: A large-scale benchmark for improved object detector pre-training
[3] An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale
[4] Vitaev2: Vision transformer advanced by exploring inductive bias for image recognition and beyond


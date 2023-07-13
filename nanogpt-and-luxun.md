---
title: nanoGPT + 鲁迅
date: 2023-02-12 23:24:02
tags:
- GPT
- AI
- Pytorch
- Python
---
## 1. 起因
今晚看到了Simon Willison 的只使用自己的博客内容来训练nanoGPT的[实验](https://til.simonwillison.net/llms/training-nanogpt-on-my-blog)，觉得挺有意思，突发奇想，能不能在鲁迅的文集上训练一个nanoGPT，然后生成很具辨识度的鲁迅风格的文字呢？由于nanoGPT结构简单，鲁迅的文集在GitHub上可以下载到，因此通过简单的代码修改加实验，就得到一个在鲁迅作品上训练的GPT2模型(无别的语料库的预训练），简单测试下，以“故乡”开头，让模型生成鲁迅风格的文字：
```plain
故乡，债是佩服的。
 我一向对于新青年的态度，先来说话，谢容易做的，然而伏园已经见过几样，感觉的是另外捧之数，以为先前的例子。今但近来做了做事，自己也还不做，不能先行通，所以生在冷静和“人生”，三妇一苦闷，觉得大约是如此隔膜
和曹操，于是非意模茶炛，可以说是太高了，所以现在便能教育，竟�如此。
 但汝实在有给法历代的，不久就在绝末年间，我想显出向大家饮一趟，而汉子大毒是怀旧的，就要贫足有打劫，可以永掠的。这种事情，中国有一个大官左翼阿，（陀思妥习），有敢请佛喜，总要适说一点�
```

还算有鲁迅文字的风格，但逻辑一窍不通，整体还是难让人满意，不知道是GPT2能力的问题还是我实验设置的问题。 Anyway，这里共享一下我实验的流程，有兴趣的朋友可以参考，进行改进。本文涉及的代码修改代码已经提交到这个[仓库了](https://github.com/vra/nanoGPT)，可以参考，文末会附上更多例子。

<!--more-->

## 2. 操作流程
### 2.1 下载nanoGPT源码并安装依赖
```bash
git clone https://github.com/karpathy/nanoGPT
cd nanoGPT
conda create --name nanogpt  python=3.9
conda activate nanogpt
pip install transformers datasets tiktoken tqdm wandb numpy httpx torch torchvision
```

### 2.2 数据预处理
进入代码目录后，重建文件夹`data/lunxun`，用于存放数据。

从[这里](https://github.com/gzx1996/luxun/blob/master/book/book.txt)下载鲁迅全集，放到`data/luxun`目录下，然后进行下面的处理：
+ 去掉所有编者加的注释(由于注释都是以`[n]`这种形式开头的，因此在VIM中可以用`0,$s/^\[.\+//g`命令来去掉)
+ 由于我们想要的是鲁迅白话文的风格，因此手动去掉所有文言文的作品和翻译作品(文言文在最开头的《坟》集子里，翻译作品在最后)
+ 去掉单行的日期文字（如`(一九一八年二月二日)`，可以在VIM中用`g/^(一九.\+/d`去掉)

我处理后的文本地址在[这里](https://github.com/vra/nanoGPT/tree/master/data/luxun/book.txt)。

然后编写代码`prepare.py`, 读取文本，构造训练集和验证集，数据比例9:1。
```python
#!/usr/bin/env python
import os
import json
import tiktoken
import numpy as np
import random

input_file_path = os.path.join(os.path.dirname(__file__), "book.txt")

entries = []
with open(input_file_path, "r") as f:
    for line in f:
        if line.strip() and len(line) > 2:
            entries.append(line)

print(f"len of lines: {len(entries)}")
# Shuffle entries
random.shuffle(entries)

n = len(entries)
train_entries = entries[: int(n * 0.9)]
val_entries = entries[int(n * 0.9):]

# Turn those into strings
train_data = " ".join("{}".format(entry) for entry in train_entries)
val_data = " ".join("{}".format(entry) for entry in val_entries)

# encode with tiktoken gpt2 bpe
enc = tiktoken.get_encoding("gpt2")
train_ids = enc.encode_ordinary(train_data)
val_ids = enc.encode_ordinary(val_data)
print(f"train has {len(train_ids):,} tokens")
print(f"val has {len(val_ids):,} tokens")

# export to bin files
train_ids = np.array(train_ids, dtype=np.uint16)
val_ids = np.array(val_ids, dtype=np.uint16)
train_ids.tofile(os.path.join(os.path.dirname(__file__), "train.bin"))
val_ids.tofile(os.path.join(os.path.dirname(__file__), "val.bin"))
```
处理好的训练验证集在[这里](https://github.com/vra/nanoGPT/tree/master/data/luxun)，可以直接使用。

### 2.3 训练网络
数据集构建完成后，就可以训练模型了。在代码库根目录，执行下面的命令:
```bash
python train.py \
  --dataset=luxun \
--compile=False \
--batch_size=8 \
--dtype=float16
```
具体训练参数可以查看`train.py`，包括训练的层数、batch size，训练后端等等。

训练的模型默认保存在`out/ckpt.pt`。

训练22000次迭代的时候我停止了实验，loss是0.15左右。

### 2.4 测试模型
测试代码在`sample.py`，默认提示词为空(`start='\n'`)可以通过添加`--start="xxx"`来修改提示词:
```bash
python sample.py --start="故乡"
```

## 3. 更多例子
### 3.1 人工智能
```plain
人工智能力。两社丈一多，西辛，是发昏了的结果，但去的四五十岁，死的陈源教授却很不通，我也就不再有了。忍不知道阿Ｑ的名字是怎么写的？这不过是一个问境。他总还拿着四个年的过头放在心里，说道，“哀，遇的。他可
是弯口，道是阿Ｑ；近来已经做了杜师长了，半年的大武人，你还和他对面具汗说：‘非常救命！’‘是情愧 夫子’的学理论矛盾的工人，……而且跳不过是这三秒……。”
 而已 偶战线告了一个大问题，拿去做的纠纷，而他们就癖在《试玈书》的第一幅，来因为又是删节的，还和所放的做。不过如果加以细见，不想多写了东�
```
### 3.2 文艺复兴
```plain
文艺复兴运动，也是指惺把文艺当承受照时代，更易于政治家，中国并不见立刻，万分折中间，更何况失了国家，只有拜读之处，这就是政治心软到新舖式的必读书。……”
 我们──由此满可知道河南的内心眼和明白的点灵魂。我在曾经想做以看空虚洋，决不叫看情形。因为我想，便可以支持生活的原因，至少，更进一步而到中国来，他们也给了世界上的美�家所指见的最多也并非精微坏，莫非看翻译
，可说是不算太多了。
 问题。
 “我们没有见过这种东西，便怎么办呢？”
 递进句也不是有许多话。
 “可以可以，”四铭吃了点
```

### 3.3 新文化
```plain
新文化运动，也许因为他们已经有了“力”这句话的责任了。在那里和他们的风化是并不相禁多的。
 阿呼呜呼兮呜呼阿呼，
 八九年
 二、浙江艳七百
 一九二五年十二月三十日风雨之夜示，此地声声流鼓近山腌至责诼谢。
 阿Ｑ的讲到文学说，他们会打断了阿Ｑ的名目退向王的头发，向公司被挤出去了。
 最末的批评，是“没有话派的书，对于政府来往往解释，加以泄除，以政治的运命，至于失败，那倒是往往会说，我非常危险。
 小娘枟不用小说的经济字的由校的文章，使是屠戮政府，是凡这些的，但我知道画家一致攻，一致的经历
```

如果本文操作中有误的地方，还请专业人士多指出讨论。

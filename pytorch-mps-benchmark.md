---
title: Pytorch Apple Silicon GPU 训练与测评
date: 2022-05-19 16:46:24
tags:
- Mac
- Pytorch
- Python
- GPU
---
今天中午看到Pytorch的官方博客发了Apple M1 芯片 GPU加速的文章，这是我期待了很久的功能，因此很兴奋，立马进行测试，结论是在MNIST上，速度与P100差不多，相比CPU提速1.7倍。当然这只是一个最简单的例子，不能反映大部分情况。这里详细记录操作的一步步流程，如果你也感兴趣，不妨自己上手一试。
<!--more-->

## 加速原理
苹果有自己的一套GPU实现API Metal，而Pytorch此次的加速就是基于Metal，具体来说，使用苹果的Metal Performance Shaders（MPS）作为PyTorch的后端，可以实现加速GPU训练。MPS后端扩展了PyTorch框架，提供了在Mac上设置和运行操作的脚本和功能。MPS通过针对每个Metal GPU系列的独特特性进行微调的内核来优化计算性能。新设备在MPS图形框架和MPS提供的调整内核上映射机器学习计算图形和基元。

因此此次新增的的device名字是`mps`，使用方式与`cuda`类似，例如：
```python
import torch
foo = torch.rand(1, 3, 224, 224).to('mps')

device = torch.device('mps')
foo = foo.to(device)
```

是不是熟悉的配方，熟悉的味道？可以说是无门槛即可上手。


此外发现，Pytorch已经支持下面这些device了，确实出乎意料:
+ cpu, cuda, ipu, xpu, mkldnn, opengl, opencl, ideep, hip, ve, ort, mps, xla, lazy, vulkan, meta, hpu


## 环境配置
为了使用这个实验特性，你需要满足下面三个条件：
1. 有一台配有Apple Silicon 系列芯片（M1, M1 Pro, M1 Pro Max, M1 Ultra)的Mac笔记本
2. 安装了**arm64**位的Python
3. 安装了最新的`nightly`版本的Pytorch 

第一个条件需要你自己来设法满足，这篇文章对它的达到没有什么帮助。

假设机器已经准备好。我们可以从[这里](https://docs.conda.io/en/latest/miniconda.html)下载arm64版本的miniconda(文件名是`Miniconda3 macOS Apple M1 64-bit bash`)，基于它安装的Python环境就是arm64位的。下载和安装Minicoda的命令如下：
```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh 
chmod +x Miniconda3-latest-MacOSX-arm64.sh 
./Miniconda3-latest-MacOSX-arm64.sh 
```
按照说明来操作即可，安装完成后，创建一个虚拟环境，通过检查`platform.uname()[4]`是不是为`arm64`来检查Python的架构:
```bash
conda config --env --set always_yes true
conda create -n try-mps python=3.8
conda activate try-mps
python -c "import platform; print(platform.uname()[4])"
```
如果最后一句命令的输出为`arm64`，说明Python版本OK，可以继续往下走了。  

第三步，安装nightly版本的Pytorch，在开启的虚拟环境中进行下面的操作：
```bash
python -m pip  install --pre torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/nightly/cpu
```
执行完成后通过下面的命令检查MPS后端是否可用:
```bash
python -c "import torch;print(torch.backends.mps.is_built())"
```
如果输出为`True`，说明MPS后端可用，可以继续往下走了。


## 跑一个MNIST
基于Pytorch官方的example中的MNIST例子，修改了来测试cpu和mps模式，代码如下:
```python mark:85
from __future__ import print_function
import argparse
import time
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
from torchvision import datasets, transforms
from torch.optim.lr_scheduler import StepLR


class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.conv1 = nn.Conv2d(1, 32, 3, 1)
        self.conv2 = nn.Conv2d(32, 64, 3, 1)
        self.dropout1 = nn.Dropout(0.25)
        self.dropout2 = nn.Dropout(0.5)
        self.fc1 = nn.Linear(9216, 128)
        self.fc2 = nn.Linear(128, 10)

    def forward(self, x):
        x = self.conv1(x)
        x = F.relu(x)
        x = self.conv2(x)
        x = F.relu(x)
        x = F.max_pool2d(x, 2)
        x = self.dropout1(x)
        x = torch.flatten(x, 1)
        x = self.fc1(x)
        x = F.relu(x)
        x = self.dropout2(x)
        x = self.fc2(x)
        output = F.log_softmax(x, dim=1)
        return output


def train(args, model, device, train_loader, optimizer, epoch):
    model.train()
    for batch_idx, (data, target) in enumerate(train_loader):
        data, target = data.to(device), target.to(device)
        optimizer.zero_grad()
        output = model(data)
        loss = F.nll_loss(output, target)
        loss.backward()
        optimizer.step()
        if batch_idx % args.log_interval == 0:
            print('Train Epoch: {} [{}/{} ({:.0f}%)]\tLoss: {:.6f}'.format(
                epoch, batch_idx * len(data), len(train_loader.dataset),
                100. * batch_idx / len(train_loader), loss.item()))
            if args.dry_run:
                break


def main():
    # Training settings
    parser = argparse.ArgumentParser(description='PyTorch MNIST Example')
    parser.add_argument('--batch-size', type=int, default=64, metavar='N',
                        help='input batch size for training (default: 64)')
    parser.add_argument('--test-batch-size', type=int, default=1000, metavar='N',
                        help='input batch size for testing (default: 1000)')
    parser.add_argument('--epochs', type=int, default=4, metavar='N',
                        help='number of epochs to train (default: 14)')
    parser.add_argument('--lr', type=float, default=1.0, metavar='LR',
                        help='learning rate (default: 1.0)')
    parser.add_argument('--gamma', type=float, default=0.7, metavar='M',
                        help='Learning rate step gamma (default: 0.7)')
    parser.add_argument('--no-cuda', action='store_true', default=False,
                        help='disables CUDA training')
    parser.add_argument('--use_gpu', action='store_true', default=False,
                        help='enable MPS')
    parser.add_argument('--dry-run', action='store_true', default=False,
                        help='quickly check a single pass')
    parser.add_argument('--seed', type=int, default=1, metavar='S',
                        help='random seed (default: 1)')
    parser.add_argument('--log-interval', type=int, default=10, metavar='N',
                        help='how many batches to wait before logging training status')
    parser.add_argument('--save-model', action='store_true', default=False,
                        help='For Saving the current Model')
    args = parser.parse_args()
    use_gpu = args.use_gpu

    torch.manual_seed(args.seed)

    device = torch.device("mps" if args.use_gpu else "cpu")

    train_kwargs = {'batch_size': args.batch_size}
    test_kwargs = {'batch_size': args.test_batch_size}
    if use_gpu:
        cuda_kwargs = {'num_workers': 1,
                       'pin_memory': True,
                       'shuffle': True}
        train_kwargs.update(cuda_kwargs)
        test_kwargs.update(cuda_kwargs)

    transform=transforms.Compose([
        transforms.ToTensor(),
        transforms.Normalize((0.1307,), (0.3081,))
        ])
    dataset1 = datasets.MNIST('../data', train=True, download=True,
                       transform=transform)
    dataset2 = datasets.MNIST('../data', train=False,
                       transform=transform)
    train_loader = torch.utils.data.DataLoader(dataset1,**train_kwargs)
    test_loader = torch.utils.data.DataLoader(dataset2, **test_kwargs)

    model = Net().to(device)
    optimizer = optim.Adadelta(model.parameters(), lr=args.lr)

    scheduler = StepLR(optimizer, step_size=1, gamma=args.gamma)
    for epoch in range(1, args.epochs + 1):
        train(args, model, device, train_loader, optimizer, epoch)
        test(model, device, test_loader)
        scheduler.step()


if __name__ == '__main__':
    t0 = time.time()
    main()
    t1 = time.time()
    print('time_cost:', t1 - t0)
```
测试CPU：
```bash
python main.py
```

测试MPS:
```bash
python main --use_gpu
```
在M1机器上测试发现，训一个Epoch的MNIST，CPU耗时33.4s，而使用MPS的话耗时19.6s，加速1.7倍，好像没官方博客中说的那么多，估计是跟模型太小有关。

我又在Nvidia P100 GPU服务器上进行了测试，CPU耗时34.2s，使用CUDA 耗时20.4s，加速比1.67倍，跟M1差不多，整体速度略低于M1。
下面是一个总结表格：

|机器|内存|CPU耗时|GPU耗时|加速比|
|-|-|--|-|-|
|M1|16G|33.4s|19.6s|1.70|
|P100|256G|34.2s|20.4s|1.67|

## 跑一下VAE模型
类似地，跑一下这个仓库里面地VAE模型，发现CPU模式正常，换成MPS后loss不断增大，最后到nan，看来还是有bug的 (毕竟是实验特性)，可以在Pytorch GitHub 仓库里面提issue，期待更好的Pytorch。
```bash
[W ParallelNative.cpp:229] Warning: Cannot set number of intraop threads after parallel work has started or after set_num_threads call when using native parallel backend (function set_num_threads)
Train Epoch: 1 [0/60000 (0%)]   Loss: 550.842529
Train Epoch: 1 [1280/60000 (2%)]        Loss: 330.613251
Train Epoch: 1 [2560/60000 (4%)]        Loss: 4705.016602
Train Epoch: 1 [3840/60000 (6%)]        Loss: 183532752.000000
...
Train Epoch: 6 [40960/60000 (68%)]      Loss: nan
Train Epoch: 6 [42240/60000 (70%)]      Loss: nan
```

## 一个愿景
开头提到，关注这个特性挺久了，其实我最初的想法，是希望一台普通计算设备（不带GPU的笔记本，智能手机）都能训非常快的模型。因为GPU卡很昂贵，只有科研机构和大公司才有，普通人购买成本比较高，而云服务商提供的GPU按时收费，价格不菲。另一方面，所有普通笔记本和智能手机都有不错的CPU，算力不错，如果能将这部分性能合理地利用起来，就像深度学习前的时代一样，有一台笔记本就能用MatLab快速地进行科学实验，这样才能将AI推广到更多人，将AI平民化，也避免了大公司在硬件资源上的垄断和显卡巨大的能耗。

今天的Mac GPU训练至少是在降低深度学习能耗和深度学习模型训练的"轻量化"上面有了一个大的进步，你可以抱着笔记本在床上训练改变AI模型了 😊 。但以Mac笔记的价格，很难说在平民化方向上有任何的进展。

---
title: talkGPT4All 2.0
date: 2023-05-27 14:49:45
tags:
- GPT
- GPT4All
- LLM
---

### 1. 概述

[talkGPT4All](https://github.com/vra/talkGPT4All)是基于[GPT4All](https://gpt4all.io/index.html)的一个语音聊天程序，运行在本地CPU上，支持Linux，Mac和Windows。它利用OpenAI的Whisper模型将用户输入的语音转换为文本，再调用GPT4All的语言模型得到回答文本，最后利用文本转语音(TTS)的程序将回答文本朗读出来。

关于 talkGPT4All 1.0的介绍在[这篇文章](https://juejin.cn/post/7217112585802498107)。

talkGPT4All 1.0的[视频效果](https://www.zhihu.com/zvideo/1625779747656515584)。

由于GPT4All一直在迭代，相比上一篇文章发布时(2023-04-10)已经有较大的更新，今天将GPT4All的一些更新同步到talkGPT4All，由于支持的模型和运行模式都有较大的变化，因此发布 talkGPT4All 2.0。

具体来说，2.0版本相比1.0有下面的更新。

首先是GPT4All框架支持的语言模型从1个增加到8个，并且可以一键切换模型。具体的模型是

*   Vicuna-7B-1.1-q4\_2
*   Vicuna-7B-1.2-q4\_2
*   wizardLM-7B.q4\_2
*   GPT4All
*   GPT4All-J
*   GPT4All-J-v1.1
*   GPT4All-J-v1.2
*   GPT4All-J-v1.3

可以看到除了GPT4All系列的模型，这个框架也支持Vicuna和Wizard的模型了。更多模型因为证书和格式的问题，还在集成中。

根据GPT4All的[文档](https://gpt4all.io/index.html)，不同模型在benchmark上的结果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/988ae9ef513049d68d790e742f9e2139~tplv-k3u1fbpfcp-zoom-1.image)


可以看到GPT4All系列的模型的指标还是比较高的。

另一个重要更新是GPT4All发布了更成熟的Python包，可以直接通过pip 来安装，因此1.0中集成的不同平台不同的GPT4All二进制包也不需要了。集成PyPI包的好处多多，既可以查看源码学习内部的实现，又更方便定位问题（之前的二进制包没法调试内部代码），且保证了不同平台安装命令一致（之前不同平台二进制包不同）。

还有一个变化是GPT4All会自动按需下载模型，因此用户不需要手动下载和维护模型路径。同时将模型统一放置到<https://gpt4all.io/models/> 目录下，测试国内模型下载速度也很快，大家玩起来也会更舒服。

核心的更新内容就这些，下面对talkGPT4All的安装和使用进行说明，后面有空会添加一些多个语言模型效果的对比视频。
<!--more-->

### 2. 安装与使用

### 2.1 安装

由于GPT4All, OpenAI Whisper 和TTS工具都是PyPI的包，因此所有的依赖都可以用pip 命令来安装。

流程大致上就是clone代码，创建Python虚拟环境，安装依赖，开始聊天：
```bash
git clone https://github.com/vra/talkGPT4All.git
cd talkGPT4All
python -m venv talkgpt4all-env
source talkgpt4all-env/bin/activate
pip install -U pip
pip install -r requirements.txt
```

如果在Linux环境下使用，还需要安装 TTS 工具 pyttsx3 的一些前置依赖，例如在Ubuntu下，可以这么安装（别的发行版切换apt 为对应的包管理命令应该就可以）：
```bash
sudo apt update && sudo apt install -y espeak ffmpeg libespeak1
```

依赖安装完后即刻开始聊天：
```bash
python chat.py
```

语音输入问题，Whisper会识别语音到文字，第一次需要下载模型Whisper的模型，可能耗时会比较久。Whisper 模型默认存储地址是`~/.cache/whisper/`。

文字识别后，输入到语言模型部分后会下载语言模型文件，文件默认存储到`~/.cache/gpt4all` 目录。

### 2.2 切换不同的LLM

默认的语言模型是GPT4All-J-v1.3,，可以通过命令行选项--gpt-model-name来切换模型，所有的选项是：

```python
"ggml-gpt4all-j-v1.3-groovy"
"ggml-gpt4all-j-v1.2-jazzy"
"ggml-gpt4all-j-v1.1-breezy"
"ggml-gpt4all-j"
"ggml-gpt4all-l13b-snoozy"
"ggml-vicuna-7b-1.1-q4_2"
"ggml-vicuna-13b-1.1-q4_2"
"ggml-wizardLM-7B.q4_2"
```

例如可以这样使用：
```bash
python chat.py --gpt-model-name ggml-wizardLM-7B.q4_2
```

如果模型未下载过，会进行下载。

这里有个小问题，GPT4All工具貌似没有对模型的完整性进行校验，所以如果之前模型下载没完成就退出，再次进入后会加载不完整的文件，造成报错。所以需要手动删除不完整的文件再次完整下载后使用。

### 2.3 切换不同大小的Whisper模型

OpenAI Whisper 也有一系列的模型，模型越大识别结果应该是越准。talkGPT4All默认使用的是base模型，也提供了命令行参数--whisper-model-type 来修改，所有的可选项是:
```python
"tiny.en"
"tiny"
"base.en"
"base"
"small.en"
"small"
"medium.en"
"medium"
"large-v1"
"large-v2"
"large"
```

例如可以这样使用：
```bash
python chat.py --whisper-model-type large
```

### 2.4 调整声音语速

talkGPT4All也提供了一个参数--voice rate 来调整 TTS发音的速度，默认是165，设置越大速度越快:
```bash
python chat.py --voice-rate 200
```

### 3. 缺陷和改进思考

其实talkGPT4All一直以来的缺陷是比较明显的：

1.  大模型在CPU上出词太慢
2.  离线的文本转语音的程序太生硬

针对第一个问题，我的思考是这样，要在非Nvidia GPU设备上流畅运行基于Transformer结构的大语言模型，除了4比特量化、fp16这种 low-hang fruit外，必须要做很多底层的AI工程的优化，这个我觉得我自己是没有能力来完成的，甚至我猜测，可能GPT4All背后的Nomic AI团队也没有这方面的积累来解决这个问题。

可喜的是最近看到了[MLC LLM](https://github.com/mlc-ai/mlc-llm)这个工作，是TVM 团队利用TVM Unity来优化语言模型，成功地将Vicuna-7B运行到了[Android](https://github.com/mlc-ai/mlc-llm/blob/main/android/README.md)和[iOS](https://github.com/mlc-ai/mlc-llm/blob/main/ios/README.md)手机上，我自己用小米12 Pro测试每秒能输出3～4个token，体验算是比较好的。这也是我第一次在自己手机上运行大语言模型，也意识到真正要提高大语言模型的覆盖设备，一个极致优化的底层AI工具是必不可少的。

所以对talkGPT4All这个项目感兴趣的朋友也可以了解一下MLC LLM这个工作，我认为在未来这个项目会促进LLM的真正落地。

针对第二个问题，说实话还没有找到比较自然的离线 TTS Python工具，如果看到这篇文章的你有这方面的经验，欢迎评论交流～


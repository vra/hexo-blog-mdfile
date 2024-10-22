---
title: talkGPT4All 2.5-更多模型以及更加真实的TTS
date: 2023-11-22 12:57:30
tags:
 - AI
 - Whisper
 - GPT4All
 - Python
 - pip
---
### 1. 概述

[talkGPT4All](https://link.zhihu.com/?target=https%3A//github.com/vra/talkGPT4All)是基于[GPT4All](https://link.zhihu.com/?target=https%3A//gpt4all.io/index.html)的一个语音聊天程序，运行在本地CPU上，支持Linux，Mac和Windows。它利用OpenAI的Whisper模型将用户输入的语音转换为文本，再调用GPT4All的语言模型得到回答文本，最后利用文本转语音(TTS)的程序将回答文本朗读出来。

今年4、5月份的时候，我发布了talkGPT4All 1.0版本和2.0版本，链接见下：

[talkGPT4All: 基于GPT4All的智能语音聊天程序](https://zhuanlan.zhihu.com/p/618826760)
[talkGPT4All 2.0:现在支持8个语言模型了](https://zhuanlan.zhihu.com/p/632592897)

大家反馈最大的问题是TTS太机械了，听着很难受（具体可以看前面两篇文章的评论区）。而最近TTS领域的进展很多，例如很受欢迎的 coqui-ai的[TTS](https://github.com/coqui-ai/TTS) 库，提供了TTS、声音克隆和声音变换的功能。上周末尝试了一下，发现内置了一些开箱即用的TTS模型，刚好可以集成到 talkGPT4All 中，解决目前采用的 [pyttsx3](https://pypi.org/project/pyttsx3/)合成声音太机械的问题。
<!--more-->

另外查看 GPT4All 的文档，从2.5.0开始，之前的.bin 格式的模型文件不再支持，只支持.gguf 格式的模型。因此我也是将上游仓库的更新合并进来，修改一下 talkGPT4All 的接口。

由于GPT4All 是从2.5.0开始不兼容.bin 格式老模型的，是一个很大的 break change。为了统一，我将更新后的 talkGPT4All 版本也命名为 2.5.0。

2.5.0版本效果视频见[这里](https://zhuanlan.zhihu.com/p/668275615)。
### 2. 如何使用

如果想直接使用的话，采用pip安装talkGPT4All包即可：

```bash
pip install talkgpt4all
```

安装完后进入聊天：
```bash
talkgpt4ll 
```

talkGPT4All 现在支持15个模型，可以通过-m 来切换你想用的GPT模型，所有模型列表见 3.2章节。
```bash
talkgpt4all -m gpt4all-13b-snoozy-q4_0.gguf
```

### 3. 实现细节

这里重点讲一下此次更新中涉及到的两个点：coqui-ai/TTS如何使用以及GPT4All 2.5.0以后如何调用GPT模型。

#### 3.1 coqui-ai/TTS使用

直接使用pip install TTS 即可安装 coqui-ai/TTS包，里面包含了很多功能，这里只简单展示如何调用一个现有的TTS模型。

首先列出所有的TTS模型：
```python
from TTS.api import TTS
print(TTS().list_models()) 
```


输出：
```bash
'tts_models/multilingual/multi-dataset/xtts_v2',
'tts_models/multilingual/multi-dataset/xtts_v1.1',
'tts_models/multilingual/multi-dataset/your_tts',
'tts_models/multilingual/multi-dataset/bark',
'tts_models/bg/cv/vits',
'tts_models/cs/cv/vits',
'tts_models/da/cv/vits',
'tts_models/et/cv/vits',
'tts_models/ga/cv/vits',
'tts_models/en/ek1/tacotron2',
'tts_models/en/ljspeech/tacotron2-DDC',
'tts_models/en/ljspeech/tacotron2-DDC_ph',
'tts_models/en/ljspeech/glow-tts',
'tts_models/en/ljspeech/speedy-speech',
'tts_models/en/ljspeech/tacotron2-DCA',
'tts_models/en/ljspeech/vits',
'tts_models/en/ljspeech/vits--neon',
'tts_models/en/ljspeech/fast_pitch',
'tts_models/en/ljspeech/overflow',
'tts_models/en/ljspeech/neural_hmm',
'tts_models/en/vctk/vits',
'tts_models/en/vctk/fast_pitch',
'tts_models/en/sam/tacotron-DDC',
'tts_models/en/blizzard2013/capacitron-t2-c50',
'tts_models/en/blizzard2013/capacitron-t2-c150_v2',
'tts_models/en/multi-dataset/tortoise-v2',
'tts_models/en/jenny/jenny',
'tts_models/es/mai/tacotron2-DDC',
'tts_models/es/css10/vits',
'tts_models/fr/mai/tacotron2-DDC',
'tts_models/fr/css10/vits',
'tts_models/uk/mai/glow-tts',
'tts_models/uk/mai/vits',
'tts_models/zh-CN/baker/tacotron2-DDC-GST',
'tts_models/nl/mai/tacotron2-DDC',
'tts_models/nl/css10/vits',
'tts_models/de/thorsten/tacotron2-DCA',
'tts_models/de/thorsten/vits',
'tts_models/de/thorsten/tacotron2-DDC',
'tts_models/de/css10/vits-neon',
'tts_models/ja/kokoro/tacotron2-DDC',
'tts_models/tr/common-voice/glow-tts',
'tts_models/it/mai_female/glow-tts',
'tts_models/it/mai_female/vits',
'tts_models/it/mai_male/glow-tts',
'tts_models/it/mai_male/vits',
'tts_models/ewe/openbible/vits',
'tts_models/hau/openbible/vits',
'tts_models/lin/openbible/vits',
'tts_models/tw_akuapem/openbible/vits',
'tts_models/tw_asante/openbible/vits',
'tts_models/yor/openbible/vits',
'tts_models/hu/css10/vits',
'tts_models/el/cv/vits',
'tts_models/fi/css10/vits',
'tts_models/hr/cv/vits',
'tts_models/lt/cv/vits',
'tts_models/lv/cv/vits',
'tts_models/mt/cv/vits',
'tts_models/pl/mai_female/vits',
'tts_models/pt/cv/vits',
'tts_models/ro/cv/vits',
'tts_models/sk/cv/vits',
'tts_models/sl/cv/vits',
'tts_models/sv/cv/vits',
'tts_models/ca/custom/vits',
'tts_models/fa/custom/glow-tts',
'tts_models/bn/custom/vits-male',
'tts_models/bn/custom/vits-female',
'tts_models/be/common-voice/glow-tts'
```


我从英文('en')的 TTS 模型中挑选了一个听起来比较好的 `tts_models/en/ljspeech/glow-tts`, 作为 talkGPT4All的默认 TTS，调用方式如下：
```python
from TTS.api import TTS

# 初始化TTS模型
tts = TTS(model_name="tts_models/en/ljspeech/glow-tts", progress_bar=False)

# 或者用离线下载的模型路径
tts = TTS(model_path="/path/to/model")

# 合成文本对应的音频并保存到文件
tts.tts_to_file(text="Hello there", file_path="hello.wav")
```


如果因为网络原因模型在Python代码中下载不了，可以手动下载模型，然后指定TTS初始化中的model_path 为模型的本地路径。

#### 3.2 GPT4All 2.5.0以后模型的调用

gguf 格式的模型目前有15个，各有特点：

![](https://picx.zhimg.com/80/v2-be3555b71a240b52bbc48865090126cc_1440w.png?source=d16d100b)


所有模型的详细信息在[这里](https://github.com/nomic-ai/gpt4all/blob/a328f9ed3fdf238835429dd45940850724d0a652/gpt4all-chat/metadata/models2.json#L145)，下面我列出所有支持的模型，方便命令行调用时参考：
```bash
mistral-7b-openorca.Q4_0.gguf
mistral-7b-instruct-v0.1.Q4_0.gguf
gpt4all-falcon-q4_0.gguf
orca-2-7b.Q4_0.gguf
orca-2-13b.Q4_0.gguf
wizardlm-13b-v1.2.Q4_0.gguf
nous-hermes-llama2-13b.Q4_0.gguf
gpt4all-13b-snoozy-q4_0.gguf
mpt-7b-chat-merges-q4_0.gguf
orca-mini-3b-gguf2-q4_0.gguf
replit-code-v1_5-3b-q4_0.gguf
starcoder-q4_0.gguf
rift-coder-v0-7b-q4_0.gguf
all-MiniLM-L6-v2-f16.gguf
em_german_mistral_v01.Q4_0.gguf
```


而 GPT4All chat 模式的调用方式也发生了变化，新版本需要这么调用：
```python
gpt_model = GPT4All("mistral-7b-openorca.Q4_0.gguf", allow_download=True)       
with gpt_model.chat_session():
    answer = gpt_model.generate(prompt="hello")
```


需要显式地创建`chat_session` context manager。

### 4. 总结

上面就是这次更新的主要内容，总的来说就是采用了更自然的TTS，更新代码以支持 GPT4All最新的break change。

欢迎大家试用、反馈bug。

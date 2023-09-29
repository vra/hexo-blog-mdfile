---
title: Code Llama 解读系列1-论文阅读
date: 2023-09-29 10:17:04
tags:
 - LLM
 - Code Llama
 - Deep Learning
 - Python
---
> Code Llama 是 Meta 基于 Llama 2 的代码生成AI模型， 在同类开源模型中取得比较好的结果。这里计划写3篇系列文章，从论文细节、代码使用、效果实测方面对 Code Llama 进行解读，欢迎关注我了解后续文章。

### 1. 背景
2023年8月24日，Meta 开源了基于 [Llama 2](https://github.com/facebookresearch/llama)) 通用 LLM 的代码生成系列模型 [Code Llama](https://github.com/facebookresearch/codellama))，支持Python, C++, Java, PHP, TypeScript, C# 和 Bash 编程语言，而且支持学术研究和商业使用。


另外 Code Llama 官方代码只提供了一些简单的使用示例，没有提供生产环境可用的 VSCode 等 工具的插件，搜索了一下也没找到简单易用的第三方开发的插件。相信很快就会有人做出来的。如果你有看到基于 Code Llama 的 VSCode 或者 Vim 插件，欢迎评论指教。
<!--more-->

一些链接：
+ 代码仓库: https://github.com/facebookresearch/codellama
+ 论文PDF: [链接](https://scontent-sjc3-1.xx.fbcdn.net/v/t39.2365-6/369856151_1754812304950972_1159666448927483931_n.pdf?_nc_cat=107&ccb=1-7&_nc_sid=3c67a6&_nc_ohc=wURKmnWKaloAX-JEHAz&_nc_ht=scontent-sjc3-1.xx&oh=00_AfBOeTPJWHrxyxjNs4TLPACB4M7xQIwQcM5SMRMzDo8uCg&oe=64EEAC4F)
+ Meta AI 博客文章：[链接](https://ai.meta.com/blog/code-llama-large-language-model-coding/)


### 2. 数据说明
这篇论文中提到了几个不同的数据，有一些数据的构造还是挺巧妙的，这里列出来，希望能给大家一些启发。

#### 2.1 2T token dataset
首先是 Llama 2 的训练数据，虽然不是这篇论文的贡献，但因为 Code Llama 模型都是从 Llama 2 初始化的，所以 这些代码生成的特化模型也都是见过这些数据的，包含它们中的知识。

Llama 2 是使用 2T token 数据训练的，其中代码相关的部分有80B token，占比只有4%。

#### 2.2 500B token dataset
这篇论文先是提出了通用的 500B tokens 数据集, 85% 都是关于代码的，所以可以认为这个500B 就是一个代码数据集。

#### 2.3 100B token Python dataset
除了通用的 500B token 代码数据集，为了提高对 Python 代码的生成能力，论文又提出了 100B token Python dataset。

#### 2.4 RLHF V5 dataset
这是 Llama 2 论文中使用的人工纠正的数据集，是为了让代码可以更好的对应回答提问者的指令，也为了生成更安全的代码（可以理解成对生成的代码中某些危险的代码进行过滤？）

#### 2.5 self-instruct 5B token dataset
self-instruct 是一个生成代码数据集，通过给llama2提问代码任务，得到它的结果，作为gt。 会不会存在错误答案？这里论文设计了一个很精巧的方案来构造生成代码数据：
1. 让 LLama 2 70B 大模型设计 Easy 和 Medium 难度的编程题，每次出50道题目，要求不重复，且可以用一个单独的Python函数来实现。总共得到52000个去重的问题。下面是提示词和一些回答的例子：
![](/imgs/code_llama_1/20230826223518.png)
2. 对上面得到的每个问题，用 Code Llama 7B 模型生成 单元测试代码和10个解题的代码，然后在解题代码上运行单元测试，将第一个通过单元测试的代码加入到 self-instruct 数据集中。

这是一种很巧妙的设计，通过单元测试来判断代码的对错，能够做到完全自动化地构造数据。

当然如果单元测试代码本身错，那可能会将错误的解题代码加入到训练集中。而根据[这篇论文](https://arxiv.org/abs/2308.02312))的分析，作为最强的 LLM，ChatGPT 生成的代码错误率为52%。所以有理由认为  Code Llama 7B 生成的单元测试代码也会有错误，因此 self-instruct 不是一个完美的数据集。

### 3. 训练策略
论文中针对不同的模型，尝试了不同的训练策略，整体来说是和数据集比较匹配的。

#### 3.1 从头训练 vs Finetune
论文中实验发现，采用通用llm (Llama 2)初始化，再在code数据集上finetune比在code数据集上从头训练效果要好。但同时也发现，只使用 Llama 2 模型来做代码生成，效果比 Llama 2 + Code 数据集训练要差，可见 2T token  pretrain + 500B token finetune才是做通用代码生成的最好选择。

#### 3.2 代码补全功能
上面的基本训练策略中只会给定前面的代码，补全或预测后面的代码，但在有些常见，是已知前面和后面的代码，给出中间的代码，比如docstring的生成，就需要知道前面的内容(函数的名字和参数)和后面的内容（函数的具体实现），才能给出比较准确的函数说明docstring。这种任务模式论文中称之为补全 (Infilling)。

这种需求跟 LLM 预测下一个 token 的任务模式是不同的，因此需要对训练模式进行改造。总体来说，论文采用了 Casual Mask 的模式来训练网络，也就是将训练序列中间的一部分移动到最后，让网络来预测这部分内容。具体来说，将训练中的token分割为前缀、中间部分和后缀部分，分割位置利用均匀分布来确定。训练时以一半的概率喂前缀-后缀-中间（PSM）格式 token 序列，一半的概率喂后缀-前缀-中间（SPM）格式的 token 序列。
#### 3.3 长上下文输入微调
Llama 2 模型的最长 token 数目为4096，对于代码生成任务来说，还是比较小，比如分析整个仓库中的代码，可能很容易超出限制。因此 Code Llama 在 finetune 阶段将 token 数从4096 提升到16384，提升了4倍。

位置embedding 采用旋转位置embedding, query 和 key vector都是 Rxn的一个线性组合，而R是一个块对角矩阵，也就是只有对角线和附近的4个值非零，每个位置i处的R公式如下：
![](/imgs/code_llama_1/20230827002151.png)

![](/imgs/code_llama_1/20230827002347.png)
d为总的token 维度。

#### 3.4 指令finetune
这部分也是为了生成更安全的代码，也更好地针对提问者的问题进行更人性化的回答。个人理解，这部分本身在策略上没有太多trick，核心是数据的构造和采集。

### 4. 模型说明
基于几种数据和几个训练策略，就能得到不同的模型。
Code Llama 系列包含三大类模型，每类模型包含 7B, 13B 和 34B 三种参数大小，共9个模型。

![](/imgs/code_llama_1/20230826222457.png)

第一类是Code Llama 通用代码生成模型，采用 Llama 2 的模型参数初始化，在 500B token 数据集上训练。其中 7B 和 13B 模型还进行了代码补全数据集上的训练，适用于 IDE 中实时的代码补全，而 34B 因为速度问题，并不适合实时补全，更适合作为编程助手。

第二类是 Code Llama-Python，这是针对 Python 专门优化的模型，在 500B 通用数据训练的基础上，又在额外的 100B Python 数据集上进行了finetune。

第三类是 Code. Llama-Instruct，在 Code Llama 通用模型基础上，增加了在 RLHF V5 和 self-instruct 数据集上的 finetune 过程，可以生成更符合指令需求的代码。

### 5. 结果对比

论文中比较了非常多测试集上的指标，太多反而不知道模型的效果到底怎么样，所以这里也不列出来了。下面放的是博客文章中的和别的模型的对比表格，反而比较简洁，可以做一个大致的对比。


![](/imgs/code_llama_1/20230826213734.png)

当然根据我之前的 LLMs 使用经验，实际使用时的智能感受貌似不能很好地和 Benchmark 上的结果对应起来，相差几个点对最终结果的提升有多大不太好说。 所以 Code Llama 具体使用体验如何，留待下一篇文章来分析。


---
title: 全世界最准确的翻译DeepL到底有多强? 一个有意思的例子
date: 2022-05-21 17:44:43
tags:
---
- Deep Learning
- Translator

在知乎上偶然看到了一个基于深度学习的翻译器DeepL，实际体验了一下，确实发现比Google Translate, 百度翻译等工具好用，因此最近抛弃了之前的翻译工具，开始往DeepL切换，毕竟在阅读英文内容的过程中还是有很多单词和词组的意思不了解。最近在阅读DeepMind的一篇文章的时候，看到一段有意思的话，对比了一下，发现DeepL真的比竞品厉害，更加加速了我抛弃之前工具的速度。具体什么例子呢，如下细说。
<!--more-->

## 一个有意思的例子
在阅读DeepMind的[这篇文章](https://www.deepmind.com/blog/from-lego-competitions-to-deepminds-robotics-lab)的时候，我发现了有一个段落里面的一句话有点看不懂，具体如下：
> My afternoons are a mix of meetings, coding and – now that most people are back in the office – an impromptu chat or two. That’s one of my favourite parts of being in the office – the random catch-ups and whiteboard sessions that help me learn and move quickly. From there I’ll take a quick snack break, and if the weather is nice, head to the balcony to catch up on some of my favourite US sports podcasts (**I still haven’t made the switch from football to *football***). Then I’ll code a little while longer.

截图如下：
![段落](/imgs/quote.png)

这里前情提要是这样的：这个科学家是个英国人，在美国读了大学，在谷歌工作了一些时间后transfer到DeepMind了。这里提到一些每日活动安排，前面说到他会看一会最喜欢的美国体育播客，然后就是红色框里面的那句`I still haven’t made the switch from football to football`，这里我没看懂，两个football是什么意思呢？依稀记得英语课上说够football有不同的含义，但不知道具体是什么。

因此我动用了翻译工具，结果如下。

谷歌翻译的结果:
![google_translate](/imgs/google_translate.jpg)

百度翻译的结果：
![baidu_translate](/imgs/baidu_translate.jpg)


看来那句话这几个翻译器都没看懂，让我也看得一头雾水。

然后尝试了一下DeepL，结果出乎意料:
![deepl_translate](/imgs/deepl_translate.jpg)
谜题解开了，这里前面的`football`是美式英语，意思是橄榄球，而后面的`football`是英式英语表达，意思是足球，这也契合了前面说的看美国体育的播客的说法，DeepL估计是从上下文推断出来的，别的翻译器看来还是在理解文本上差一些。

至此我对DeepL的敬畏又增加了几分。

## 一个疑问
看了下DeepL网站的介绍，确实做了很多创新。但我在想，像谷歌怎么厉害的公司，也有财力和物力来做相同的事情，为什么他们没有做或者说做不到跟DeepL那么好呢？

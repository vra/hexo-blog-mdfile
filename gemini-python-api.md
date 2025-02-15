---
title: 谷歌Gemini和Gemma大模型的Python调用
date: 2024-08-29 08:30:32
tags:
 - AI
 - Google
 - Gemini
 - Gemma
 - Python
 - LLM 
 - Deep Learning
---

## 1. 说明
Google 发布了Python 包google-generativeai，可以方便地调用Gemini和Gemma 系列的模型，免费模型只需要申请一个Key，无需任何费用。
![](/imgs/gemini-python-api/gemini-1.png)

而且Gemini 1.5 Pro模型还支持一些多模态任务，例如检测bbox，实际测试下来效果还不错。
这里简单写一个流程，体验效果。
<!--more-->

## 2. key获取与包安装
访问Google AIStudio 来进行Key注册：[Google AI Studio](https://aistudio.google.com/app/prompts/new_chat)
Python包安装:
```bash
pip install -U google-generativeai 
```

## 3. 文本输入 
简单使用大模型的对话能力，例如讲一个鬼故事：
```python
# pip install -U google-generativeai
import google.generativeai as genai
import os
import PIL.Image

# obtain your key at https://aistudio.google.com/
genai.configure(api_key=os.environ["GOOGLE_API_KEY"])
model = genai.GenerativeModel('gemini-1.0-pro-latest')
response = model.generate_content("讲一个鬼故事")
print(response.text)
```
输出结果:
![](/imgs/gemini-python-api/gemini-2.png)

最后一句有点惊悚…

## 4. 多模态输入
随便找了一张跳舞的人的图片，测试一下人体框检测效果，这里使用Gemini-1.5-pro来多模态检测人体框：

prompt如下：'Return bounding boxes of the <object>, in the format of [ymin, xmin, ymax, xmax] 
```python
# pip install -U google-generativeai
import google.generativeai as genai
import os
import PIL.Image

# obtain your key at https://aistudio.google.com/
genai.configure(api_key=os.environ["GOOGLE_API_KEY"])
model = genai.GenerativeModel('gemini-1.5-pro-latest')

# output bbox
img = PIL.Image.open("dancer.jpg")
prompt = 'Return bounding boxes of the dancer, in the format of [ymin, xmin, ymax, xmax]'
response = model.generate_content([img, prompt])
print(response.text)
```
检测结果:
![](/imgs/gemini-python-api/gemini-3.png)

## 5. 参考
1. [google-generativeai · PyPI](https://pypi.org/project/google-generativeai/)
2. [Building a tool showing how Gemini Pro can return bounding boxes for objects in images (simonwillison.net)](https://simonwillison.net/2024/Aug/26/gemini-bounding-box-visualization/)
3. [Explore vision capabilities with the Gemini API  |  Google AI for Developers](https://ai.google.dev/gemini-api/docs/vision?lang=python#bbox)

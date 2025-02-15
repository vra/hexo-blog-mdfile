---
title: GPT实用功能之润色README
date: 2024-07-28 08:18:25
tags:
  - GPT
  - GPT-4o
  - LLM
  - AI
  - README
  - Markdown
---
今天发现GPT-4o对于GitHub Readme文档的润色还是很不错的，很自动添加一些花里胡哨的功能，看起来很fancy。
<!--more-->

具体的提示词如下：

```plain
将下面github 仓库的readme进行细化重写，添加更多fancy的标记和文本，使得更加详细和吸引人
实际测试效果对比：
```

润色前：
````markdown
# mimic_head 
Unofficial One-click Version of LivePortrait, with Webcam Support

## Features
+ with webcam, video and single image support
+ with cpu, mps and cuda backend support, you can run it without Nvidia GPU!


## Screenshot
+ Image mode:
![mimic_head image mode screenshot](assets/imgs/img_demo.jpg)

+ Video mode:


https://github.com/user-attachments/assets/1aef9ae6-7d05-4fea-a03c-2c3de76df8b1



+ Webcam mode:
NOTE: FPS ~= 13 on my mac laptop and there is observable delay in this video


https://github.com/user-attachments/assets/6a2ce4c5-e3f2-40cd-9fe9-c081407aaca1


## Install and use
```bash
pip install mimic_head 
mimic_head run
```
````


润色后：
````markdown
# 🎭 mimic_head 

![Project Image](https://img.shields.io/badge/Maintained-yes-green.svg)
![License: MIT](https://img.shields.io/github/license/vra/mimic_head)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/vra/mimic_head)

🚀 **Unofficial One-click Version of LivePortrait** with Webcam Support!

## 🌟 Features
- 📷 **Webcam, Video, and Single Image Support**:
  - Easily switch between different input modes to suit your needs.
- 🖥️ **CPU, MPS, and CUDA Backend Support**:
  - Run seamlessly without needing an Nvidia GPU!

## 📸 Screenshot
### Image Mode:
![mimic_head image mode screenshot](assets/imgs/img_demo.jpg)

### Video Mode:
https://github.com/user-attachments/assets/1aef9ae6-7d05-4fea-a03c-2c3de76df8b1

### Webcam Mode:
**Note: FPS ~ 13 on a Mac laptop with noticeable delay.**

https://github.com/user-attachments/assets/6a2ce4c5-e3f2-40cd-9fe9-c081407aaca1


## 🚀 Getting Started

### 📦 Installation
To install and use `mimic_head`, simply run the following command:

```bash
pip install mimic_head
```

### 🛠️ Usage
Once installed, you can start the application by running:

```bash
mimic_head run
```

## 📚 Documentation

For detailed instructions and advanced usage, please refer to our [README](https://github.com/vra/mimic_head).

## 🤝 Contributing
We welcome contributions! If you'd like to contribute, please fork the repository and use a feature branch. Pull requests are warmly welcomed.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 🛡️ License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 💬 Contact
For any inquiries, questions, or issues, please open an issue in this repository or contact me at <wyf.brz@gmail.com>.

## 📝 Acknowledgments
- Special thanks to the original creators of LivePortrait for their work.
- Inspired by the amazing community contributions and ideas.

## ⭐ Support
If you like this project, please give it a ⭐ on [GitHub](https://github.com/vra/mimic_head)!

---

Made with ❤️ by [Yunfeng Wang](https://github.com/vra).
````

可以看到，自动添加了：

+ 项目徽章：添加了一些项目徽章（例如维护状态和许可证），使得README.md看起来更专业。
+ 标题和说明：使用表情符号和强调文本使标题和说明更具吸引力。
+ Features：详细描述了项目的主要功能，并添加了适当的表情符号来增强视觉效果。
+ Screenshot：各个模式下的截图分别展示，并链接到对应的视频。
+ Getting Started：以更加详细和有条理的方式提供安装和使用说明。
+ Documentation：提供了指向详细文档的链接。
+ Contributing：提供了详细的贡献指南，鼓励用户参与。
+ License：明确项目的许可证信息。
+ Contact：提供联系信息。
+ Acknowledgments：感谢原始创作者和社区对项目的贡献。
+ Support：鼓励用户给项目打星。

看上去专业了很多，算是很实用的工具了。
```



---
title: 使用 Python 88 行代码写一个简易的 Android AI 程序
date: 2023-10-14 23:22:03
tags:
 - LeptonAI
 - Python
 - Android
 - AI
---

TL;DR:
我基于 LeptonAI 和 Beeware Python 库，利用 88 行的Python，不用写一行Java代码，在手机上做了一个 SDXL text-to-image 的Demo，效果见[这里](https://zhuanlan.zhihu.com/p/661358058)的视频。

作为一个爱折腾写Python比较多的人，我一直在想一个事情：能否将熟悉的Python技术栈的能力带到移动平台中，不用写哪些繁琐的Native开发代码，就能在移动端跑起来一个AI Demo呢？因为相比PC，移动端设备的用户数多得多，每个人都有一台手机，但并不是每个人都有一台电脑。

一次偶然的机会，我发现了 [Beeware](https://beeware.org/)，一个目标 “Write once. Deploy everywhere.“ 的跨平台 Python 工具箱。基于 Beeware 工具箱写的 Python 程序可以在 PC，Web，Android 和 iOS 上运行，因此正是我想要的。

一切听起来很美好，但实际使用时也遇到很多问题。
<!--more-->

首先是 Beeware 在移动端支持的 Python 包有限，比如像对 Pytorch 的支持就有问题 (可以import但运行时报错)，所以手机本地没法直接运行 Pytorch AI模型，至少我没有跑通。

另一个是 Beeware 工具链中的 GUI 库 toga 太简单了，一些复杂的功能实现不了，比如网络推理时加一个显示在窗口最顶层的转圈的特效。所以只能做一些比较toy的小的项目，没法做真正可以用的产品。

所以不想写繁琐的 Natvie代码的话，另一个选择可能就是写 基于小程序的 Web 代码了，至少小程序的UI功能还是很齐全的。

Anyway，虽然有这些约束，但还是可以用 Beeware 做一些简单的 Python Demo，比如这里我就结合 [LeptonAI](https://www.lepton.ai/)和 Beeware，一行 Android 开发的都不用写，总共利用 88 行的 Python 代码，做出来了一个简单的 SDXL text-to-image Android 端 Demo。

首先说说一下服务端。SDXL 部署在 LeptonAI 的云平台上，提供公网可访问的 AI 服务。关于 LeptonAI 的使用和 SDXL 的部署，可以参考我这篇[文章](https://zhuanlan.zhihu.com/p/661243511)。简单来说安装 LeptonAI Python SDK 后，使用下面的三条命令创建模型镜像，然后在 LeptonAI 的云平台进行部署:

```bash
# 创建镜像
lep photon create --name sdxl --model hf:hotshotco/SDXL-512

# 登录云平台
lep login -c xxx:xxxxxxxxxxxxxxxx

# 推送镜像到云平台
lep photon push --name sdxl
```

客户端就是这个App， 整体功能很简陋，用户在输入框填入提示词，点击生成图片的按钮后，代码读取用户输入，构造网络请求，然后将 text-to-image 生成的图像返回给客户端，客户端进行解析后再展示。

开发流程是先在 Mac 上调试代码，成功后再进行一些微调，就能跑到手机上。

具体来说，整个过程中用到的 Beeware 命令如下：
```bash
# 交互式地构建项目目录
briefcase new

# 在Mac上调试代码
briefcase dev

# 创建 Android 开发环境，会自动在命令行下载NDK等
briefcase create android 

# 编译代码，生成 APK文件
briefcase build android
```
`briefcase` 是 Beeware 工具箱中用来将 Python 代码转换为 Native 应用的工具。

在 Mac 上运行正常，往手机上微调过程中，也有一些细节要注意。

首先是需要将依赖包写入到`pyproject.toml`中的`requires` 字段中，Mac上可能因为已经提前安装了一些第三方包而在使用时没有报错，但在移动端使用时需要将所有用到的包都加入到apk中。
    
由于 Beeware 貌似不支持 requests 包，所以需要将 比较简洁的 requests 请求方式修改为基于系统库的`urllib.request` 请求方式。

由于Android环境没有环境变量，因此需要将原先代码中读取环境变量中的TOKEN的代码去掉，这里采用了不太科学的方法，直接将TOKEN写死在代码中。

Python 代码更新有时候不会生效，需要手动删除 Build 目录再执行 `briefcase build android`的命令。

最后也将 88 行代码列出来，完整代码仓库在[这里](https://github.com/vra/sdxl-python-app)，感兴趣的小伙伴可以自己玩玩。


```python
"""
An Application based on Python and LeptonAI!
"""
import json
import io
import os
import urllib.request

from PIL import Image as PIL_Image

import toga
from toga.style import Pack
from toga.style.pack import COLUMN, ROW


class AISDK:
    def __init__(self):
        # Android 端没法用环境变量，这里只能将 TOKEN 写死在代码中
        api_token = "xxxxxxxxxxxx"
        self.url = "https://xxx-sdxl-deploy.bjz.edr.lepton.ai/run"
        self.headers = {
            "Content-Type": "application/json",
            "accept": "image/png",
            "Authorization": f"Bearer {api_token}",
        }

    def process(self, prompt, img_save_path):
        print("ai processing begin...")
        data = {"num_inference_steps": 25, "prompt": prompt, "seed": 42}
        req = urllib.request.Request(self.url, headers=self.headers, data=json.dumps(data).encode('utf-8'))
        response = urllib.request.urlopen(req)
        res = response.read()

        image_data = io.BytesIO(res)
        image = PIL_Image.open(image_data)
        image.save(img_save_path)
        print("ai processing done")


class SDXLApp(toga.App):
    def startup(self):
        self.sdk = AISDK()
        self.img_save_path = os.path.join(os.path.dirname(__file__), "aigc_img.jpg")

        main_box = toga.Box(style=Pack(direction=COLUMN))

        name_label = toga.Label("Your prompt: ", style=Pack(padding=(0, 5)))
        self.name_input = toga.TextInput(style=Pack(flex=1))

        name_box = toga.Box(style=Pack(direction=ROW, padding=5))
        name_box.add(name_label)
        name_box.add(self.name_input)

        button = toga.Button(
            "Generate Image", on_press=self.run_aigc, style=Pack(padding=5)
        )

        main_box.add(name_box)
        main_box.add(button)

        print(self.img_save_path)
        self.image = toga.Image(self.img_save_path)
        self.image_view = toga.ImageView(self.image)

        self.main_window = toga.MainWindow(title=self.formal_name)
        self.main_window.content = main_box
        self.main_window.content.add(self.image_view)
        self.main_window.show()

    def run_aigc(self, widget):
        # 清除已有结果
        self.main_window.content.remove(self.image_view)
        self.image_view = toga.ImageView(image=None)

        prompt = self.name_input.value
        self.sdk.process(prompt, self.img_save_path)

        image = toga.Image(self.img_save_path)
        self.image_view = toga.ImageView(image)
        self.main_window.content.add(self.image_view)


def main():
    return SDXLApp()


if __name__ == "__main__":
    SDXLApp()
```



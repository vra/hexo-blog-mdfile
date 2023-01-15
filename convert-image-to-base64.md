---
title: 用Python将图片转换为base64字符串
date: 2022-10-07 22:19:37
tags:
- Python
- HTML
- OpenCV
---
### 1. 概述
无他，这篇博文记录一下利用Python将OpenCV图片转换为base64字符串并在网页上进行展示的过程，权当备忘。可在[这里](https://github.com/vra/image-to-base64)查看源码。
<!--more-->

### 2. Show the code
```py
import base64

import cv2


def img_to_base64(img_path):
    img = cv2.imread(img_path)

    _, buffer = cv2.imencode('.jpg', img)
    text = base64.b64encode(buffer).decode('ascii')
    return text


def create_html_file(text, file_name):
    html_pattern = """
    <html>
    <body>
    <img src="data:image/png;base64,{}"/>
    </body>
    </html>
    """

    html = html_pattern.format(text)
    with open(file_name, 'w') as f:
        f.write(html)


if __name__ == '__main__':
    img_path = 'data/cat.jpg'
    html_file_name = 'data/show_img.html'

    text = img_to_base64(img_path)
    create_html_file(text, html_file_name)
```

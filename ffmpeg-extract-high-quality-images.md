---
title: ffmpeg抽取高清图像帧
date: 2023-01-15 22:27:33
tags:
- 图像
- ffmpeg
---

使用[ffmpeg](https://ffmpeg.org)可以方便地从视频中抽取图像帧：
```bash
ffmpeg -i /path/to/video.mp4 image-folder/%06d.jpg
```

但实际测试发现，抽取的图像帧比较模糊，有明显的块效应。

搜索时有人说可以加`-q:v 1 -qmin 1 -qmax 1`来提高图像质量
```bash
ffmpeg -i /path/to/video.mp4 -q:v 1 -qmin 1 -qmax 1 image-folder/%06d.jpg
```
测试发现确实有一些提升，但还是能看到明显的模糊。

最后发现，把抽取的图像格式从`.jpg`修改为`.png`，结果就是高清且无块效应的了：
```bash
ffmpeg -i /path/to/video.mp4 image-folder/%06d.png
```
另外PNG格式的图像存储大小要大一些，但不会太大，还是能接受的。

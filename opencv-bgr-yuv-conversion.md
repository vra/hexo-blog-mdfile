---
title: OpenCV Code Snippets——BGR与YUV转换
date: 2019-12-19 23:16:39
tags:
 - OpenCV
 - Python
 - C++
 - Linux
 - 总结
---
## 概述
OpenCV BGR 图 转 YUV 图的代码，网上没有比较完整的示例，使用的时候搜索比较费劲。这里写一个代码片段和例子，方便查找。
<!--more-->

## C++ 代码
在 Ubuntu 16.04 自己从源码编译的OpenCV 4.1.0 上测试通过，具体如下：
```cpp
// file name: convert.cpp
#include <opencv2/opencv.hpp>

// BGR 转 YUV
void BGR2YUV(const cv::Mat bgrImg, cv::Mat &y, cv::Mat &u, cv::Mat &v) {
    cv::Mat out;
    cv::cvtColor(bgrImg, out, cv::COLOR_BGR2YUV);
bgr
    cv::bgr channel[3];
    cv::split(out, channel);

    y = channel[0];
    u = channel[1];
    v = channel[2];
}

// YUV 转 BGR
void YUV2BGR(const cv::Mat y, const cv::Mat u, const cv::Mat v, cv::Mat& bgrImg) {
    std::vector<cv::Mat> inChannels;
    inChannels.push_back(y);
    inChannels.push_back(u);
    inChannels.push_back(v);

    // 合并3个单独的 channel 进一个矩阵
    cv::Mat yuvImg;
    cv::merge(inChannels, yuvImg);

    cv::cvtColor(yuvImg, bgrImg, cv::COLOR_YUV2BGR);
}

// 使用例子
int main() {
    cv::Mat origImg = cv::imread("test.png");

    cv::Mat y, u, v;
    BGR2YUV(origImg, y, u, v);

    cv::Mat bgrImg;
    YUV2BGR(y, u, v, bgrImg);

    cv::imshow("origImg", origImg);
    cv::imshow("Y channel", y);
    cv::imshow("U channel", u);
    cv::imshow("V channel", v);
    cv::imshow("converted bgrImg", bgrImg);
    cv::waitKey(0);

    return 0;
}
```
在命令行用下面的命令来运行，查看代码有无问题：
```bash
g++ -o convert convert.cpp  --std=c++11 `pkg-config --cflags --libs opencv`
./convert
```

## Python 实现
在 `pip` 安装的 OpenCV 4.1.2 上测试通过，具体如下：
```python
# file name: convert.py
import cv2


def bgr2yuv(img):
    yuv_img = cv2.cvtColor(img, cv2.COLOR_BGR2YUV)
    y, u, v = cv2.split(yuv_img)

    return y, u, v


def yuv2bgr(y, u, v):
    yuv_img = cv2.merge([y, u, v])
    bgr_img = cv2.cvtColor(yuv_img, cv2.COLOR_YUV2BGR)

    return bgr_img


def main():
    orig_img = cv2.imread('test.png')

    y, u, v = bgr2yuv(orig_img)

    bgr_img = yuv2bgr(y, u, v)

    cv2.imshow('orig_img', orig_img)
    cv2.imshow('Y channel', y)
    cv2.imshow('U channel', u)
    cv2.imshow('V channel', v)
    cv2.imshow('bgr_img', bgr_img)
    cv2.waitKey(0)


if __name__ == '__main__':
    main()
```
通过下面的命令来执行：
```bash
python3 convert.py
```

我的博客即将同步至腾讯云+社区，邀请大家一同入驻：https://cloud.tencent.com/developer/support-plan?invite_code=3obyzp09lomco

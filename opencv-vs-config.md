title: OpenCV配置——在Visual Studio中使用OpenCV
id: 454
categories:
  - 学习总结
  - 计算机视觉
  - OpenCV 
date: 2015-04-22 23:01:24
tags:
---

[OpenCV](http://opencv.org/)是图像领域经常会用到的工具库函数的集合，有C/C++,Java和Python等语言的接口，并且适用于Windows，Linux，Mac OS桌面开发平台和Android 和IOS移动开发平台。目前已经出了1.x系列和2.x系列，3.0 Beta版也已经出了。OpenCV配置起来还是挺费事的，虽然网上已经有很多很全面也很有用的参考文章，我还是打算把自己配置的过程写下来，以后肯定还会配置这个东西，希望到时候有个方便的参考。

这篇文章记录在Windows平台上，如何安装OpenCV并且在Visual Studio 的C/C++开发环境中使用之。

我用的是Windows 7，Visual tudio 2012 Ultimate。

<!--more-->

## 下载OpenCV包

在[opencv下载](http://opencv.org/downloads.html) 页面上，下载想要安装的版本。据说3.x系列会修改较多的API名称等，所以建议下载比较新的版本。我下的是2.4.10。下载之后将文件解压。

解压后会看到看到两个文件夹：`build`和`source`，`build`文件夹下面是已经编译好的库文件和可执行文件，而`source`文件夹下面是未编译的源文件。我们在写程序时用到的是一些编译好的lib和dll文件，所以只要在程序中添加了头文件，调用了相应的函数，然程序运行时能找到相应的库文件（包括动态库文件即`.dll`文件和静态库文件，即`.lib`文件）就可以了。所以针对这个情况，大概做以下几个步骤就可以了。

##  添加环境变量


添加环境变量是为了让程序在运行时能找到函数对应的动态链接库（`dll`）。**要注意的是，OpenCV对于32位程序和64位程序有不同的dll目录，并且对于不同的版本的VS，也有不同的dll文件目录。**

在`build`目录下，`x86`下面包含了32位程序所需的`dll`文件，`x64`目录下面包含了64位程序所需`dll`文件。在这个两个目录下，都有`vc10`,`vc11`,`vc12`三个文件夹，分别是针对`vs2010`,`vs2012`和`vs2013`。为了使32位程序和64位程序都能编写通过，我一般将两者目录下的和VS版本对应的文件夹下的`bin`目录都加入PATH变量中。所以在PATH环境变量中增加如下内容：

```bash
;D:\program_file\opencv\build\x86\vc11\bin;D:\program_file\opencv\build\x64\vc11\bin
```
其中build前面的位置是我安装opencv的目录，安装位置不同前面部分也应改为相应的目录。

## 生成独立的OpenCV配置属性表

我们的目标是通过操作生成一个单独的OpenCV配置属性表，然后将其导出保存起来，将来在需要用到OpenCV的程序中，直接导入这个保存的属性表即可。
下面几步都是在VS开发环境里面进行。

 1. 创建一个空项目，通过视图->属性管理器找到属性管理器页面。每个项目都可以有四个编译情况，分别是：`Debug|win32`、`Release|win32`、`Debug|x64`、`Release|x64`，基本步骤都类似，下面针对`Debug|win32`来说。
 2. 在`Debug|win32`文件夹上右击，选择添加新项目属性表，在弹出的对话框里，给这个表取名为OpenCV_Debug_32.props，然后点击添加。
 3. 双击新建的属性表，就会弹出熟悉的MFC复古风格的属性设置页了。
 4. 在属性页上，点击C/C++->常规->附加库包含目录，在这里添加OpenCV安装路径下的include目录，具体如下：

 ```bash
 D:\program_file\opencv\build\include
 ```
 同样的，build前面是opencv的安装路径，按实际情况选择。

 5. 在属性页上，点击链接器->常规->附加库目录，在这里添加OpenCV安装路径下的lib目录。注意：对不同编译情况和不同版本的VS，lib文件夹目录不同。对于VS2012下面的`Debug|win32`模式，lib文件夹目录为：

```bash
D:\program_file\opencv\build\x86\vc11\lib
```

其中`x86`目录表示是针对`win32`的，`vc11`表示是适用于`VS2012`的。

 6. 在属性页上，点击链接器->输入->附加依赖项，在里面添加附加依赖的lib文件：

 ```bash
 opencv_calib3d2411d.lib
 opencv_contrib2411d.lib
 opencv_core2411d.lib
 opencv_features2d2411d.lib
 opencv_flann2411d.lib
 opencv_gpu2411d.lib
 opencv_highgui2411d.lib
 opencv_imgproc2411d.lib
 opencv_legacy2411d.lib
 opencv_ml2411d.lib
 opencv_nonfree2411d.lib
 opencv_objdetect2411d.lib
 opencv_ocl2411d.lib
 opencv_photo2411d.lib
 opencv_stitching2411d.lib
 opencv_superres2411d.lib
 opencv_ts2411d.lib
 opencv_video2411d.lib
 opencv_videostab2411d.lib
 ```
 注意：对于不同的版本，要将后面的2411改为相应的版本号；对于Debug版本，后面有个字母d,而对于Release版本，则没有d，应根据实际情况添加。
 7. 添加好之后，点击属性页面板右下角的应用，确定。
 8. 在`Debug|win32`文件夹上右击，选择保存，该属性表就保存好了。
 9. 在该项目目录下面找到这个属性表，保存到一个安全的地方，下次在要用OpenCV的工程里，找出属性管理器，右键，选择添加现有属性表即可。
我将四种情况所需的属性表和添加的附加依赖库的列表都放到了github上，或许能帮到你（注意只适用于VS2012）。


整个配置过程就是这样了，配置好之后就可以安心的使用OpenCV 了！

最后，测试一下，做个图像处理的“Hello World”:

```bash
#include <iostream>
#include <string>
#include <opencv2\opencv.hpp>

using namespace std;
using namespace cv;

int main()
{
	Mat img = imread("lena.jpg");

	if (img.empty())
	{
		cout << "error";
		return -1;
	}

	imshow("lena", img);
	waitKey();

	return 0;
}
```

出来的图像：

![](https://vra.blog.ustc.edu.cn/wp-content/uploads/2015/04/lena.png)

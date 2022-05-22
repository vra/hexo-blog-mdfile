---
title: Mac上如何运行OpenGL:第一个例子
date: 2021-11-20 08:33:37
tags:
 - C++
 - macOS
 - OpenGL
---
## 概述
搜索发现，OpengGL在mac下其实运行还是比较容易的，这里做一个简单的总结。
<!--more-->

## 依赖安装
安装依赖项:
```bash
brew install glfw3 glew cmake
```

## 编写OpenGL代码
编写OpenGL代码：
```cpp
 <iostream>
#include <GL/glew.h>
#include <GLFW/glfw3.h>


int main()
{
    GLFWwindow* window;

    /* Initialize the library */
    if (!glfwInit())
        return -1;

    /* Create a windowed mode window and its OpenGL context */
    window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
    if (!window)
    {
        glfwTerminate();
        return -1;
    }

    /* Make the window's context current */
    glfwMakeContextCurrent(window);

    /* Loop until the user closes the window */
    while (!glfwWindowShouldClose(window))
    {
        /* Render here */
        /* Swap front and back buffers */
        glfwSwapBuffers(window);
        /* Poll for and process events */
        glfwPollEvents();
    }

    glfwTerminate();

   return 0;
} 
```
## 编写CMake 配置文件

为了简单可复现，这里我们直接编写`CMakeLists.txt`, 内容如下：
```cmake
cmake_minimum_required(VERSION 3.10)
project(opengl_first_example)

include_directories(/usr/local/include)
link_directories(/usr/local/Cellar/glew/2.2.0_1/lib)
link_directories(/usr/local/Cellar/glfw/3.3.4/lib)

add_executable(${PROJECT_NAME} main.cpp)

target_link_libraries(${PROJECT_NAME} GLEW GLFW)
```

**需要修改其中第5行和第6行路径中的glew和glfw为你自己电脑安装的版本**

## 编译执行代码
编译代码，使用CMake的常规流程:
```bash
mkdir build
cd build
cmake .. 
make -j8 
```
编译完成后运行生成的可执行文件：
```bash
./opengl_first_example
```
可以看到一个图窗弹出来，说明OpenGL调用成功了.

## 参考
1. <https://zhuanlan.zhihu.com/p/153550789>


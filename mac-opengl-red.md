---
title: Mac OpenGL入门：显示颜色
date: 2021-11-21 16:18:23
tags:
 - OpenGL
 - C++
 - Mac
---
## 概述
这里以显示一个红色的窗口为例，展示Mac下运行OpenGL代码的一些配置项。这里采用c++ 和cmake来编译代码的方式，比用xcode更直观。
<!--more-->

## 依赖安装
```bash
brew install glfw3 glew cmake

```

## 源代码
C++源码如下：
```cpp
#include <GL/glew.h>
#include <GLFW/glfw3.h>

#include <iostream>


using namespace std;

void init(GLFWwindow* window) {
}

void display(GLFWwindow* window, double currentTime) {
    glClearColor(1.0, 0.0, 0.0, 1.0);
    glClear(GL_COLOR_BUFFER_BIT);
}

int main() {
    if (!glfwInit()) {
        exit(EXIT_FAILURE);
    }

    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 1);
	
	// mac增加的代码
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);

    GLFWwindow* window = glfwCreateWindow(600, 600, "Chapter 2 - program1", NULL, NULL);
    glfwMakeContextCurrent(window);

    if (glewInit() != GLEW_OK) {
        exit(EXIT_FAILURE);
    }

    glfwSwapInterval(1);

    init(window);

    while (!glfwWindowShouldClose(window)) {
        display(window, glfwGetTime());

        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    glfwDestroyWindow(window);
    glfwTerminate();
    
    exit(EXIT_SUCCESS);
}
```

## cmake文件
cmake 代码：

```cmake
cmake_minimum_required(VERSION 3.10)
project(show_box)

include_directories(/usr/local/include)
link_directories(/usr/local/Cellar/glew/2.2.0_1/lib)
link_directories(/usr/local/Cellar/glfw/3.3.4/lib)

add_executable(${PROJECT_NAME} ch2.1.cpp)

target_link_libraries(${PROJECT_NAME}
GLEW
GLFW
"-framework OpenGL"
)

```

## 编译代码
```bash
mkdir build
cd build
cmake ..
make -j8
./show_box
```

---
title: C++的RAII到底指的是什么
date: 2022-05-19 16:43:00
tags:
- C++
---
RAII，全称 Resource Acquisition Is Initialization，中文翻译为资源获取即初始化。这是C++中一个比较不直观的术语，而RAII的缩写也时不时遇到，总给人一种很高深但不易掌握的感觉。实际上查了资料后发现，RAII这个技术的含义其实比较明确，这里简单汇总一下从资料中的得到的知识点。
<!--more-->

## 什么是资源
这里的资源 (Resource) 是C++编程中的一个概念，表示哪些不能无限申请的变量（常有明确的含义），比如一段内存，数据库句柄，Socket，打开的文件，线程等。
个人理解，一般的内置类型变量如`int` 变量不算是资源。

## 为什么要设计 RAII 这项技术？
简单来说，RAII 这项技术的目的是将资源的生命周期绑定到某个对象（Object）上。对象，一般情况是某个类的示例。这么做有下面几个好处：
1. 保证资源在使用的时候已经进行了初始化，避免访问未初始化的内存地址而crash
2. 保证资源在程序正常退出的时候进行了释放，避免未释放导致的内存泄漏
3. 保证资源在运行出错的时候也能被正常释放

## 具体如何实现RAII？
RAII 的实现可以总结为：
+ 将每个资源封装到一个类中，类的构造函数获取资源，如果获取资源失败，则抛出一个异常。
+ 类的解构函数释放资源，并且保证不抛出异常，因此保证资源的释放是没问题的

## 一个例子
从[这里](https://docs.microsoft.com/en-us/cpp/cpp/object-lifetime-and-resource-management-modern-cpp?redirectedfrom=MSDN&view=msvc-170)拿过来的一个例子:
```cpp
class widget
{
private:
    int* data;
public:
    widget(const int size) { data = new int[size]; } // 资源获取
    ~widget() { delete[] data; } // 资源释放
    void do_something() {}
};

void functionUsingWidget() {
    widget w(1000000);   // lifetime automatically tied to enclosing scope
                        // constructs w, including the w.data member
    w.do_something();

} // automatic destruction and deallocation for w and w.data
```
这里`widget`就是一个RAII类，它将`data`这个资源绑定到类上面，在构造和析构函数里面进行资源获取和释放。



## Reference
1. <https://en.cppreference.com/w/cpp/language/raii>
2. <https://stackoverflow.com/questions/2321511/what-is-meant-by-resource-acquisition-is-initialization-raii?answertab=scoredesc#tab-top>
3. <https://docs.microsoft.com/en-us/cpp/cpp/object-lifetime-and-resource-management-modern-cpp?redirectedfrom=MSDN&view=msvc-170>

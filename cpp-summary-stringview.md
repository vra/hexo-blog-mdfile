---
title: C++中的string_view
date: 2021-01-23 15:40:39
tags:
- C++
- C++17
---

C++17标准库里面引入了轻量级的只读字符串表示类型`string_view`，用来替代`const char*` 和`const string&`，在传入函数的时候减小内存开销(因为`string_view`类只包含字符串的指针和字符串的长度值，开销小于`string`类型)。
<!--more-->

`string_view` 定义在头文件`<string_view>`中。

具体来说，C++17里面引入了模板类`basic_string_view`类，而`string_view`是针对`char`特化的类，如头文件中所表示的：
```cpp
using string_view = basic_string_view<char>;
using u8string_view = basic_string_view<char8_t>;
using u16string_view = basic_string_view<char16_t>;
using u32string_view = basic_string_view<char32_t>;
using wstring_view   = basic_string_view<wchar_t>;
```
可以看到针对不同类型的字符数组，都有对应的只读view。
顺便提一下，上述代码中用到的`using`用法是C++11引入的类型重定义（type alias)，可以给类型和函数起别名，下面是官方给的示例用法:
```cpp
#include <string>
#include <ios>
#include <type_traits>
 
// type alias, identical to
// typedef std::ios_base::fmtflags flags;
using flags = std::ios_base::fmtflags;
// the name 'flags' now denotes a type:
flags fl = std::ios_base::dec;
 
// type alias, identical to
// typedef void (*func)(int, int);
using func = void (*) (int, int);
// the name 'func' now denotes a pointer to function:
void example(int, int) {}
func f = example;
 
// alias template
template<class T>
using ptr = T*; 
// the name 'ptr<T>' is now an alias for pointer to T
ptr<int> x;
```

`string_view` 使用方法与`string`一样，而且可以由`string`类型对象相互初始化，如下所示：
```cpp
std::string_view sv1("hello world");
std::string s1(sv1);
std::string_view sv2(s1);
```

实际测试发现，相同的字符串，`string_view` 对象的大小确实比`string`对象要小，比如下面的例子：
```cpp
#include <iostream>
#include <string_view>

int main() {
	std::string_view sv1("hello world");
	std::string s1(sv1);

	std::cout << "size of string_view: " << sizeof(sv1) << std::endl;
	std::cout << "size of string: " << sizeof(s1) << std::endl;

	return 0;
}
```
在32位的机器下(x86)，输出如下：
```bash
size of string_view: 8
size of string: 28
```
因为`string_view` 只包含一个指向字符串的指针(*)和一个表示数组大小的整型数值(`int`)，因此总大小是4+4=8。而`string`是容器类型，内部结构我不太清楚，看输出整体是要比`string_view`大挺多的。

如果想在C++11的环境下使用C++17才引入的`string_view`，可以使用谷歌推出的[absl库](https://github.com/abseil/abseil-cpp)，这个库在C++11的环境下实现了很多C++14，17甚至20里面才提出的新特性，可以尝试一下。
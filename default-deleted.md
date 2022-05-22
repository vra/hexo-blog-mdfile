title: c++11新特性：default和delete 
date: 2016-01-17 10:30:39
tags:
  - 学习总结
  - C++
---
## 缘起
今早在美国的本科室友问了我下面的C++代码是什么意思：  
```cpp
#ifdef  _CV_H
#define _CV_H
class cv{
	cv(const cv&) = delete;
	cv& operator=(const cv&) = delete;

	cv(cv&&);
	cv& operator=(cv&&);
};
#endif
```
什么，`delete`居然还有这种神奇的用法？我确实以前没看过。所以我跑到实验室，自己查了些资料，大概明白这些代码是个什么意思了，所以记录下来。  
<!--more-->
## default和delete
在C++03的标准里面，如果程序代码里面没有写默认构造函数(像`cv();`)、复制构造函数、复制赋值函数(像`cv cv2=cv1;`)和析构函数，则编译器会自动添加这些函数。当程序里面写了构造函数的时候，编译器就不会自动添加默认构造函数了。    
那如果我想让一个类的实例不能通过复制构造函数来生成，该怎么办呢？一般的方法是将复制构造函数和复制赋值函数声明为`private`，而且不去具体实现它们，这样就达到了目的。  
但这样做其实是很tricky的方式，相当于利用c++的一些特性碰巧来实现，总感觉不是正确的方法。  
C++11里面可以用`default`来指定使用默认的构造函数，而且可以通过`delete`来显式地禁止一些方法，如复制构造函数和复制赋值操作，如下例： 
```cpp
struct NonCopyable{
	NonCopyable() = default;
	NonCopyable(const NonCopyable&) = delete;
	NonCopyable& operator=(const NonCopyable&) = delete;
};
```
这个例子里面，第一条语句是强制编译器生成默认构造函数作为struct的构造函数；第2、3条语句就是显式地禁用复制构造函数和复制赋值函数。  

## move constructor
既然禁止了复制构造函数，那么如果想通过已经生成的类的实例来初始化一个同类的实例，要怎么操作呢？显然，`cv cv2(cv1)`和`cv cv2=cv1;`是不可以用的了，因为复制构造函数已经被禁止了。 
C++11新定义了一个叫做`move constructor`的构造函数，签名方法如下：
```cpp
class_name(class_name &&);
class_name& operator=(class_name &&);
```
调用时这样用：
```cpp
class_name c1;
class_name c2=std::move(c1);
class_name c3(std:move(c1));
```
所谓`move`，我的理解就是类似于指针一样的概念，move生成的新的实例和原先的实例是由同一个指针指向的，即实际上是同一个实例。而且`&&`这个符号让人联想到了`**`，可能也是这个意思吧。 
这就是这个新特性的简单介绍，感觉应用场合不是很多，可能是我还没搞懂的原因吧。  
看了[这篇博客](http://blog.csdn.net/luotuo44/article/details/46779063),发现这个新特性还是很强大的啊～还是too young。 
从[这里](https://msdn.microsoft.com/en-us/library/hh567368.aspx#defaultedanddeleted)看到，vs2012里面还不支持这个特性，vs2013才开始支持。在g++中，可以通过使用`-std=c++11`来启用这个特性(我用的是g++4.9.2,默认是开启的)。  

参考链接:
<http://blog.csdn.net/pongba/article/details/1684519>
<https://en.wikipedia.org/wiki/C%2B%2B11#Explicitly_defaulted_and_deleted_special_member_functions>
<http://en.cppreference.com/w/cpp/language/move_constructor>
<http://stackoverflow.com/questions/7421825/c11-features-in-visual-studio-2012>
<http://stackoverflow.com/questions/6077143/disable-copy-constructor>


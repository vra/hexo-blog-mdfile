title: C++学习总结1——几个基本概念
id: 284
categories:
  - 学习总结
  - C++
date: 2015-01-24 17:15:14
tags:
  - 学习总结
  - C++
---

最近我在做毕设。写程序的时候，总是被C++里面的指针搞得头昏脑胀。刚开始的时候还有些浮躁，不想静下心来仔细看看指针使用的细节。过了几天发现只在Visual Studio里面调试怎么也搞不定，只好硬着头皮，重新学习指针的用法。在看书和看别人写的博客后，感觉学到了许多新的东西，不光是关于指针，还有其他一些以前我不太清楚的内容。这些知识如果不常用或不记录下来的话，肯定会忘掉的，所以我就把它们都写下来，避免以后犯同样的错误。

<!--more-->

##声明和定义

###声明(declaration)

声明用于向编译器表明变量，函数或类的类型和名字，并不会为其申请存储空间，只是向程序表明了这个对象的存在。变量声明格式如下：

```cpp
extern int i;    //变量声明

int add(int a,int b);      //函数声明

class MyClass;   //类的声明
```

注意：语句` int i;`是定义，在前面加了“extern”关键字后才是声明。

声明不会分配存储空间，所以同一个对象可以声明多次。


###定义(definition)

变量定义会为其分配存储空间，函数定义则必须给出函数实现的细节，类的定义需要指定类的成员，类函数的实现等等。定义也是一种声明（平时我们说的“声明”，特指那些不是定义的声明）。定义格式如下：

```cpp
int i;    //变量定义

int fun1(int a,int b)    //函数定义
{
    return a+b;
}

class MyClass            //类的定义
{
    public:

        void set_a(int a){m_a=a;}

        int get_a(){return m_a;}

    private:

        int m_a;
};
```

在一个程序中，每个对象只能定义一次。如果多次定义，会出现**重复定义（redefinition）**的错误。

如果声明时有初始化式，则该声明也是定义。根据“定义只能由一次，声明可以有多次”的规则，有如下例子：

```cpp
extern double pi=3.14159;    //ok:definition

doulbe pi;                   //error: redefinition of pi

extern double pi;            //ok:declaration of pi;

extern double pi=3.14;       //error: redefinition of pi
```

仔细理解上述4个语句，应该就会对声明和定义有个比较清楚的概念。

##初始化和赋值

###初始化

初始化指**创建对象的时候**给它赋初始值。如

```cpp
int age=22;      

float height;
```

则age为经过初始化的变量，height为未初始化的变量。指针类型变量的初始化过程如下：

```cpp
int age = 22;

int* p_age = &age;    //initation of p_age

int** pp_age = &p_age; //initation of pp_age
```

指针的初始化很容易犯错，像下面的错误我就犯过很多次：

```cpp
#include<iostream>

using namespace std;

int main()
{
    int* pi;

    *pi=23;           //错误：pi未初始化

    float* pf=NULL;

    pf=3.4;          //错误：pf指向不合法内存

    char* pc=NULL;

    char c='a';

    pc= &c;            //正确

    return 0;
}
```

上面例子中，pi未进行初始化，所以pi指向的是未知的内存区域，在编译会出错。像pi这样，指向内存区域不确定或无意义的指针称为“野指针”。

pf虽然经过了初始化，但指向的是内存空间的0位置，而不是指向一个float型变量的内存区域，所以运行时会出错，如下所示：

![](https://vra.blog.ustc.edu.cn/wp-content/uploads/2015/01/error_pointer_no_initation.png)

pc的使用方式则是合法的。

指针还可以用于new和delete语句，后面会进行描述。

###赋值

赋值指擦除对象的当前值并用新值来代替。可以认为，初始化就是给变量第一次赋值的过程。**对于未初始化的变量，除了用作赋值操作的左操作数，用于其他用途都是没有意义的。**

##系统默认初始化规则

所谓系统默认初始化规则，就是在声明变量时未对其进行初始化的情况下，编译器对其赋值的一套规则。对于内置类型和类类型，规则不同；对于函数内变量和函数外变量，定义规则也不同。

###内置类型变量

内置类型指int，float，char和void等基本类型（在C++中，string不是内置类型）。对于内置类型，如果在函数中定义，则系统不对其进行自动赋值；如果在函数外定义（即全局变量），则将其初始化为0（这里的“0”对不同的类型有不同的意义：对int变量，为整数0，对char变量，为‘’）。建议对每个内置类型的变量都显式地初始化。

对全局变量和局部变量的默认初始化规则不同，归根结底是因为它们保存的位置不同。全局变量保存在全局数据区，该区域的变量在编译时会自动初始化；对于局部变量，系统启动时不会为其开辟内存空间，只有当它所在的函数被调用时，才在栈中建立函数数据空间。变量如果没有显式初始化，则其值为随机值。很重要的一点是：永远不要依赖随机值。

###类类型变量

对于类类型变量，不论其是在函数内还是函数外定义，只要有默认构造函数，则系统就会自动调用其默认构造函数，如

```cpp
#include<iostream>

using namespace std;

string out;

int main()
{
    string in;    

    return 0;
}
```

因为string类有默认构造函数，所以out和in都被自动初始化为""。

如果没有默认构造函数，则定义时必须提供显式的初始化式。因为C++中类会自动地增加一个默认构造函数，所以这种情况比较少见。


##参考内容

1.《C++ Primer第4版》

2.[Declare vs Define in C and C++](http://www.cprogramming.com/declare_vs_define.html)

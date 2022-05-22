title: C++学习总结3——动态创建对象及其撤销
id: 339
categories:
  - 学习总结
  - C++
date: 2015-01-25 15:36:25
tags:
  - 学习总结
  - C++
---

这里的动态创建对象，特指在程序中通过new命令创建对象；而撤销，特指通过delete命令来删除对象并释放其内存空间。

## new和delete的基本用法

`new`命令会在堆区域分配创建一个对象，而后返回此对象的地址。
`delete`命令会释放指针指向的对象所占用的内存空间，而此后指针指向的地址是没有意义的，为了避免错误，一般来说，应该在delete后立即将指针置为NULL。

```cpp
int *pi=new int;    //动态创建对象

    //....

delete pi;    //删除动态创建对象

pi=NULL;    //将指针置为NULL
```

注意：`delete`命令只能用来释放由`new`获得的指针，而且`new`得到的指针必须通过`delete`释放掉，否则会有内存泄漏的问题。

<!--more-->

## 动态创建对象的默认初始化

用`new`创建的对象的默认初始化规则与局部变量的初始化规则相同，即：对内置类型，不进行初始化；对于类类型变量，用默认构造函数进行初始化。

举个例子：

```cpp
#include<iostream>

using namespace std;

class MyClass    //类定义
{
public:
	MyClass();
};

MyClass::MyClass()    //默认构造函数
{
	cout<<"MyClass-默认构造函数"<<endl;
}

int main()
{
	int *pi=new int;

	cout<<"*pi="<<*pi<<endl;

	MyClass *pClass=new MyClass;

	return 0;
}

```

程序运行结果为：

```cpp

*pi=-842150451

MyClass-默认构造函数

```

可见对pi 没有进行初始化，对pClass调用了MyClass类的默认构造函数。所以对于内置类型，用new申请变量时必须进行初始化，否则其指向的值是未知的。


## new int; or new int(); ？

这两种形式，有没有区别？哪一种对？到底该用哪一种呢？一开始我也搞不清楚，慢慢查查才明白其中区别。

new int()这种形式叫值初始化（value-initialize），与动态创建的不同：对于内置类型，动态创建不会对其进行初始化；而值初始化会进行初始化。对于类类型变量，两种形式都会调用默认构造函数，所以没有区别。

如下例：

```cpp
#include<iostream>

using namespace std;

class MyClass    //类定义
{
public:
	MyClass();
	~MyClass();

private:

};

MyClass::MyClass()    //默认构造函数
{
	cout<<"MyClass-默认构造函数"<<endl;
}

MyClass::~MyClass()
{
}
int main()
{
	int *p1=new int;

	int *p2=new int();

	int *p3=new int(23);

	cout<<"*p1="<<*p1<<endl;

	cout<<"*p2="<<*p2<<endl;

	cout<<"*p3="<<*p3<<endl;

	MyClass *pClass1=new MyClass;

	MyClass *pClass2=new MyClass();

    //delete 动态创建对象

	delete pClass2;

	delete pClass1;

	delete p3;

	delete p2;

	delete p1;

	return 0;
}
```

程序运行结果为

```cpp
*p1=-842150451

*p2=0

*p3=23

MyClass-默认构造函数

MyClass-默认构造函数

```

由结果可推得上述结论。

对内置类型，建议使用值初始化，可以使用特定的值对其进行初始化，如上例中p3的初始化。对于类类型，一般用第一种。


## delete or delete[] ？ 

其实这两者的区别是很明显的， 前者是释放一个位置，而后者是释放一个数组，一段位置。在使用delete[]时，编译器会获取被释放对象new时申请的数据大小size，然后全部释放size个数据。可以认为，用new申请的，用delete释放；用new[]申请的，用delete[]释放。

```cpp
#include<iostream>

using namespace std;

int main()
{

	int *pi=new int(23);

	float *pf=new float[256];

	string *pstring=new string[128];

	//...

	delete pi;    //正确

	delete[] pf;    //正确

	delete pstring;    //错误

	return 0;
}

```

在上面例子中，pstring释放的格式是错误的，相当于是只释放了pstring[0],后面的127个对象都没有被正确释放。


## 指针数组与指针的指针

指针数组的每个成员是指针，相当于是一系列指针的集合；而指针的指针就是字面意思所表示的，指向指针的指针。

这两者其实是有关系的：因为数组名相当于一个指针，所以指针数组可以看作指针的指针+特定内存空间。某些情况下，两者可以混用。

举个例子:

```cpp
#include<iostream>

using namespace std;

const int STUDENT_NUM=1024;

struct Student    //Student结构体定义
{
	int number;
	int age;
};

int main()
{
	Student **ppStudent=new Student*[STUDENT_NUM];    //指针数组，ppStudent是指针的指针

	for(int i=0;i<STUDENT_NUM;i++)  //初始化每个指针
	{
		ppStudent[i]=new Student;

		ppStudent[i]->number=i;

		ppStudent[i]->age=22;

	}

	//...

	//delete 

	for(int i=0;i<STUDENT_NUM;i++)    //释放每个指针所指的空间
	{
		delete ppStudent[i];
	}

	delete[] ppStudent;    //释放指针的指针

	return 0;
}

```

可以看到ppStudent是指针的指针，但其初始化时却用的时指针的数组。

**很容易忽略的一点是：在释放时忘记释放每个ppStudent[i]。为了避免这种错误，可以强制性地对应new[]和delete[]，new和delete，有new[]必有对应的delete[]，有new必有对应的delete。**


## 单向链表的创建和释放

链表的动态创建和释放也很容易犯错，写程序时需要多关注有关new和delete的细节部分。
下面是一个不带头结点的链表的动态创建和释放：

```cpp
#include<iostream>

using namespace std;

struct Node
{
	int data;

	Node* next;

};

int main()
{
	Node *list=new Node;    //不带头结点的链表

	list->data=0;

	list->next=NULL;

	//构造链表
	for(int i=1;i<1000000;i++)
	{

		Node *pNode=list;

		while(pNode->next!=NULL)    //找到链表尾巴
		{
			pNode=pNode->next;
		}

		pNode->next=new Node;    //在尾部增加新节点

		pNode->next->data=i;

		pNode->next->next=NULL;

	}

	//...

	//删除链表
	if(list!=NULL)
	{
		while(list->next!=NULL)
		{
			Node *p=list;

			list=list->next;

			delete p;
		}
	}

	return 0;
}

```

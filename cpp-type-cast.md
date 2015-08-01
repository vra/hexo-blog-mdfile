title: C++学习总结4——类型转换
id: 378
categories:
  - 学习总结
  - C++
date: 2015-03-31 19:52:17
tags:
  - 学习总结
  - C++
---

在写程序的时候有时会遇到类型转换的问题，而这些问题的解答每次都记不住，每次都得上网查找，经常下来，也觉得很浪费时间。所以这里我把C语言和C++里面一些常用的类型转换方式写下来，一方面为了以后查找方便，另一方面也是希望通过敲一遍能尽可能地记住转换的思路。所有这些转换的代码我已经放到了[github](https://github.com/vra/cast-utilities)上，或许可以帮到你。
<!--more-->


##几种字符串之间的转换

###字符串类型介绍

这里说的“字符串”包括`string`，'wstring'，'CString'。`string`是C++里面默认的字符串表示形式,`string`的实现使用了容器的概念，所以`string`类对象也有`begin()`，`end()`这些迭代方法。'wstring' 是保存宽字符（wide character，C++中有wchar_t类型来表示宽字符）的字符串。字符串常量在初始化'wstring'类型对象时，前面要加“L”，用以表明是宽字符串。'CString'是Windows平台下的特定的字符串，在MFC程序中使用广泛，但也可以在非MFC程序中使用，只要包括相应的头文件即可:'CString'在afx.h中定义，所以只需在程序中include <afx.h>就可以使用'CString'啦。

```cpp

#include <iostream>
#include<string>

#define _AFXDLL _AFXDLL_H_ //注意：必须写在afx.h之前

#include <afx.h>

using namespace std;

int main()
{
    string str_exampe = “This is oridnary string”;

    //注意：字符串常量前面要加L
    wstring wstr_exampe = L”This is wide string”;

    CString cstr_example = “This is Cstring”;

    return 0;
}
```

要强调的是，\_AFXDLL的定义必须写在#include<afx.h>之前，否则会出现\_AFXDLL未定义的错误。


###转换代码

CString 可以用来表示所有字符，根据字符编码的不同，可以表示宽字符或者非宽字符。Windows使用了LPCTSTR来表示你的字符是否使用了UNICODE, 如果你的程序定义了UNICODE或者其他相关的宏，那么这个字符或者字符串将被作为UNICODE字符串，否则就是标准的ANSI字符串。贴代码：


```cpp

#include <iostream>
#include<string>

#define _AFXDLL XXX //注意：必须写在afx.h之前

#include <afx.h>

using namespace std;

int main()
{

    //1-1.string to wstring
    string name = “Aldex”;
    wstring w_name(name.begin(), name.end());

    //1-2\. wstring to string
    wstring w_name2 =L”Smitch Hill”;
    string name2(w_name2.begin(), w_name2.end());
    cout << name2 << endl;

    //2-1.string to CString
    string name3 = “Cook Book”;
    CString c_name3(name3.c_str());
    cout << c_name3 << endl;

    //2-2.CString to string
    CString c_name4 = “Malon Balendo”;
    string name4 = (LPCTSTR)c_name4;
    cout << name4 << endl;

    //3-1.wstring to CString
    wstring w_name5 = L”Odlely Herben”;
    CString c_name5 = w_name5.c_str();
    cout << c_name5 << endl;

    //3-2.Cstring to wstring
    CString c_name6 = “Jephp Phoo”;

    //in ANSI build
    wstring w_name6 =CStringW(c_name6);
    //in Unicode build
    wstring w_name6 = c_name6;

    return 0;
}

```

需要强调的是，从CString转换到wstring时，需要根据当前项目的编码方式来决定该用哪种转换方法（我在VS里面试了一下，默认是ANSI 环境）。


##字符数组和字符串之间的转换

###const char\* 和char\*之间转换（const wchar_t\* 与 wchar_t\* 类似）

由于指针和数组相似的性质，下面统一用指针来陈述。

`const char*` 是常字符数组，相比`char*`，其内容是不可变的，所以从`char*` 到`const char*`是“从宽到窄”，正常可以进行，甚至不需要类型转换；而从`const char*` 到`char*`则是“从窄到宽”，转换被认为是不正常的，所以如果需要这样的转换，请先考虑程序设计是否有问题。当然，转换方式还是有的：可以用`strdup` 或者`_strdup`函数。

```cpp

#include <iostream>
#include<string>

using namespace std;
int main()
{

    //1-1.char* to const char*
    char* arr_name =“Hello, Blub Bulb “;
    const char* c_arr_name = arr_name;
    printf(“%s”, c_arr_name);

    //1-2.const char* to char
    const char* c_arr_name2 = “Android Lollipop”;

    char* arr_name2 = _strdup(c_arr_name2); //ISO C++ onformant name
    char* arr_name2 = strdup(c_arr_name2); //The Posix name

    printf(“%s”, arr_name2);

    return 0;
}

```

###char\*和wchar_t\*之间的转换

`char*`和`wchar_t*`之间的转换我很少用到，这里还是从网上找了出来，列举如下：

```cpp
#include <iostream>
#include <string>

using namespace std;

int main()
{

    //2-1.char* to wchar_t*
    char* arr_name = “Hulu Gulu”;
    size_t size = strlen(arr_name) + 1;
    wchar_t* w_arr_name = new wchar_t[size];
    mbstowcs(w_arr_name, arr_name, size);

    //2-2\. wchar_t* to char*
    wchar_t* w_arr_name2 = L”Big Ben”;
    char* arr_name2 = new char[wcslen(w_arr_name2)]; // wcslen用来求宽体字符数组的长度
    wcstombs(arr_name2, w_arr_name2, wcslen(w_arr_name2));

    return 0;
}

```

###char\* 和 string的转换（wchar_t\* 和 wstring转换同理）

`char*` 转化为`string`时会进行默认类型转换，即不需要显式地转换。而`string`转换为`const char*` 比较容易，要转换为`char*`比较麻烦，要进行内存的复制，如下：

```cpp

#include <iostream>
#include<string>

using namespace std;
int main()
{

    //3-1.char* to string
    char* arr_name = “This is my name”;
    string name(arr_name);
    cout << arr_name;

    //3-2.string to char*
    string name2 = “Hoop Hope”;

    //转换为const char* 类型
    const char* c_arr_name2 = name2.c_str();   

    //转换为char*类型
    char* arr_name2 = new char[name2.length() + 1];
    strcpy(arr_name2, name2.c_str());

    return 0;
}
```

###wchar_t\*和string，char\* 和wstring之间的转换
这一类的转换我没遇到过，但我想利用前面的这些转换方法，通过使用一个中间格式，可以完成转换，所以就再没有查这部分的转换。


##字符串和别的数据类型之间的转换

这部分总结下字符串类型和int，float这些类型转换时的一些方法。

###char\* 和int，float类型转换
这方面有三种选择：`atoi`（对float类型是`atof`）， `sscanf`和`strtol`（对float类型，是`strtof`）。[StackOverFlow上的这个回答](http://stackoverflow.com/questions/3420629/convert-string-to-integer-sscanf-or-atoi)详细的解释了三者的区别，总体来说`atoi`速度最快，但出错时没有提示，`sscanf`可以通过类似`scanf`的方式来读取，`strtol`最安全，错误提示也多，但默认是将`char*` 转换为`long int`(函数名的含义：`str to long`)。三种代码如下：

```cpp
#include <iostream>
#include<string>
using namespace std;

int main()
{
     //1-1.char* to int
     char* arr_number = “2015”;

     //a.use atoi
     int number1 = atoi(arr_number);
     cout << number1<< endl;

     //b.use sscanf
     int number2;
     sscanf(arr_number, “%d”, &number2);
     cout << number2 << endl;

     //c.use strtol
     char* pEnd; //用以指向末尾位置

     int number3 = strtol(arr_number, &pEnd, 0);//最后一个参数表示从char*的第几个位置开始读取

     //1-2.int to char*

     int time = 2345;

     //a.use itoa
     char* arr_time;
     itoa(time, arr_time, 0);//最后一个参数表示从int的第几个位置开始读取

     //b.use sprintf
     char* arr_time2;
     sprintf(arr_time2, “%d”, time);

     return 0;
}
```

###string 和int，float类型之间的转换

`string`和`int`，`float`之间的转换要用到 `stringstream`，或者`ostringstream`和`istringstream`。  

区别是`stringstream`既可以传入，也可以传出，所以既可以将`string`转化为`int`或`float`，也可以将`int`或`float`转换为`string`；而`ostringstream`只能输出`string`，所以只能将`int`或`float`转换为`string`，`istringstream`刚好反过来了。  

总的来说`stringstream`是双向的，而`i/ostringstream`是单向的。相应地，`wstring`和`int`/`float` 可以通过`wstring`或者`wostringstream`和`wistringstream`来转换。 

注意需要包含`sstream`头文件。

```cpp
#include <iostream>
#include<sstream>
#include<string>

using namespace std;

int main()
{

     //2-1.string to int
     string str_age = “73”;

     //a.use stringstream
     stringstream s(str_age);
     int age;
     s >> age;
     cout << age << endl;

     //b.use istringstream
     istringstream is(str_age);

     int age2;
     is >> age2;
     cout << age2 << endl;

     //2-2.int to string

     //a.use stringstream
     int year = 2015;
     stringstream s2;
     s2 << year;
     string str_year;
     s2 >> str_year;
     cout << str_year << endl;

     //b.use ostringstream
     ostringstream os;
     os << year;
     string str_year2 = os.str();
     cout << str_year2 << endl;

     return 0;
}

```

##参考内容

基本都是StackOverFlow的链接……

1.[http://stackoverflow.com/questions/2573834/c-convert-string-or-char-to-wstring-or-wchar-t](http://stackoverflow.com/questions/2573834/c-convert-string-or-char-to-wstring-or-wchar-t)

2.[http://stackoverflow.com/questions/4804298/how-to-convert-wstring-into-string](http://stackoverflow.com/questions/4804298/how-to-convert-wstring-into-string)

3.[http://stackoverflow.com/questions/15333259/c-stdwstring-to-stdstring-quick-and-dirty-conversion-for-use-as-key-in](http://stackoverflow.com/questions/15333259/c-stdwstring-to-stdstring-quick-and-dirty-conversion-for-use-as-key-in)

4.[http://stackoverflow.com/questions/11821491/converting-string-to-cstring-in-c](http://stackoverflow.com/questions/11821491/converting-string-to-cstring-in-c)

5.[http://stackoverflow.com/questions/2041241/convert-cstring-to-stdwstring](http://stackoverflow.com/questions/2041241/convert-cstring-to-stdwstring)

6.[http://stackoverflow.com/questions/258050/how-to-convert-cstring-and-stdstring-stdwstring-to-each-other](http://stackoverflow.com/questions/258050/how-to-convert-cstring-and-stdstring-stdwstring-to-each-other)

7.[http://stackoverflow.com/questions/6117270/mfc-stdstring-vs-cstring](http://stackoverflow.com/questions/6117270/mfc-stdstring-vs-cstring)

8.[http://stackoverflow.com/questions/2041241/convert-cstring-to-stdwstring](http://stackoverflow.com/questions/2041241/convert-cstring-to-stdwstring)

9.[http://stackoverflow.com/questions/2259544/is-wchar-t-needed-for-unicode-support](http://stackoverflow.com/questions/2259544/is-wchar-t-needed-for-unicode-support)

10.[http://baike.baidu.com/view/998109.htm](http://baike.baidu.com/view/998109.htm)

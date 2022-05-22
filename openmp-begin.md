title: OpenMP并行编程简介
date: 2016-06-17 21:49:56
tags:
 - 并行计算
 - C/C++
 - OpenMP
---

在这学期的并行计算课程中，老师讲了OpenMP,MPI，CUDA这3种并行计算编程模型，我打算把相关的知识点记录下来，便于以后用到的时候查阅。  

![](http://www.openmp.org/wp-content/uploads/openmp-menu-logo.jpg)

<!--more-->
## 概述
OpenMP是基于共享存储体系的基于线程的并行编程模型。一个共享存储的进程由多个线程组成，而OpenMP就是基于已有线程的共享编程范例。  
在OpenMP中，线程的并行化是由编程人员控制的，不是自动编程模型，而是外部变成模型。  
OpenMP采用**Fork-Join**并行执行模型。即程序开始于一个单独的主线程，主线程会一直串行地执行，遇到第一个并行域，通过如下过程完成并行操作：  
 1. Fork: 主线程创建一系列并行的线程，由这些线程来完成并行域的代码。  
 2. 当所有并行线程完成代码的执行后，它们或被同步或被中断，最后只剩下主线程在执行。


那么并行代码块是如何创建的呢？在OpenMP中，通过编译制导语句（即像`#pragma`开头的语句）来构造并行域，在原本的串行代码中，在可并行代码块周围添加编译制导语句并修改相应的代码，就可以完成并行的功能。  
运行OpenMP代码不需要安装任何额外的库或工具，标准的C/C++代码编译器执行环境就可以执行。  
下面是一个简单的OpenMP的例子：
```c
//file name: test_openmp.c
#include <stdio.h>
#include <omp.h>

int main(int argc, char** argv)
{
    int num_thread = 4;

	omp_set_num_threads(num_thread);
	#pragma omp parallel
	{
		int id = omp_get_thread_num();
		printf("hello from thread%d\n",id);
	}

    return 0;
}
```

通过`gcc --openmp test_openmp.c`来编译，运行生成的可执行文件，得到结果如下：
```shell 
hello from thread0
hello from thread3
hello from thread1
hello from thread2
```
可以看到，各个线程执行的顺序是无序的。  

## 核心知识
下面记录使用OpenMP的一些核心点。  
 1. 包含头文件`omp.h`
 2. 所有并行块由`#pragma omp`开头的编译制导语句来开始，在代码块周围要有大括号
 3. 常见的编译制导语句有`#pragma omp prallel`, 表示最基本的循环
 4. `#pragma omp parallel for`:并行部分包含一个for循环;
 5. `#pragma omp critical`:并行部分的代码一次只能由一个线程执行，相当于取消了并行化
 6. `#pragma omp barrier`: 同步并行线程，让线程等待，直到所有的线程都执行到该行
 7. `#pragma omp section`: 将并行块内部的代码划分给线程组中的各个线程，一般会在内部嵌套几个独立的`section`语句，可以使用`nowait`来停止等待 
 8. 通过`omp_set_num_threads`函数来手动设置线程数。可以看到线程数是在程序编写过程中指定的
 9. 通过`omp_get_thread_num`来获取当前线程的编号
 10. 通过`omp_get_num_threads`来获取线程总数

## 一个例子
这里举一个更完善的例子来说明。  
```cpp
#include <iostream>
#include <stdio.h>
#include <time.h>
#include <stdlib.h>
#include <math.h>
#include <omp.h>
#include <sys/time.h>

int main(int argc, char** argv)
{
	struct timeval start, end;
	gettimeofday(&start, NULL);

    if (argc != 3)
    {
        std::cout << "USAGE: num_primer <num_of_thread> <integer>" << std::endl;
        return -1;
    }

    int num_thread = atoi(argv[1]);
    int n = atoi(argv[2]);

	std::cout << "num of thread: " << num_thread << std::endl;
	std::cout << " n: " << n << std::endl;
    int* num_primer = new int[num_thread];
	for (int i = 0; i < num_thread; ++i)
	{
		num_primer[i] = 0;
	}

	omp_set_num_threads(num_thread);
	#pragma omp parallel shared(n, num_primer)
	{
		int id = omp_get_thread_num();
		
    	for (int i = id + 2; i < n + 1; i = i + num_thread)
    	{
			bool has_factor = false;
			#pragma omp parallel shared(n, i, num_primer, has_factor)
			{
				for (int j = 2; j < int(sqrt(i)) + 1; ++j)
				{
					if (i % j == 0)
					{
						has_factor = true;
						break;
					}
				}
				if (!has_factor)
				{
					++num_primer[id];
					std::cout << "id: "<< id << ", primer:" << i << std::endl;
				}
			}//pragma
    	}
	}//pragma
	
	//add all primers
	int sum_num_primer = 0;
	for (int i = 0; i < num_thread; ++i)
	{
		sum_num_primer += num_primer[i];	
	}

	std::cout << "The number of primers between 0 and " << n << " is: " << sum_num_primer << std::endl;

	gettimeofday(&end, NULL);
	double time_gap = (end.tv_sec - start.tv_sec) * 1000000u + end.tv_usec - start.tv_usec;
	printf("Time cost: %.2lf s.\n", time_gap / 100000);

    return 0;
}

```

## 参考文献
并行计算——结构，算法，编程（第3版），陈国良

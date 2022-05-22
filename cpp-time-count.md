---
title: C++ 耗时统计代码片段
date: 2021-09-04 11:44:38
tags:
 - C++
 - 总结
 - Linux
---
C++ 耗时统计代码片段

```cpp
#include <iostream>
#include <chrono>

typedef std::chrono::milliseconds ms;
using clk = std::chrono::system_clock;

void do_my_work() {
    // work code here

}

int main() {
	auto begin_time = clk::now();
    do_my_work();
	auto end_time = clk::now();

	auto duration_nn = std::chrono::duration_cast<ms>(end_time - begin_time);
	std::cout << "timecost: " << (double)duration_nn.count() << " ms" << std::endl;
	return 0;
}
```
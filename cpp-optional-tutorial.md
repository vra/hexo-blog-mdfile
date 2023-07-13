---
title: C++ std::optional 使用教程
date: 2023-06-30 22:39:41
tags:
- C++
---
### 1. std::optional 是什么


C++ 17 引入了std::optional，表示一个可能有值的对象（没有值时就是默认的`std::nullopt`)，例如这个例子中，std::optional 对象 even_value，如果`is_even` 为真的话就是128，否则就是默认值`std::nullopt`: 
```cpp
#include <iostream>
#include <optiona>

bool is_even = true;

// 在 没有值的情况下 std::optional 对象的值为 std::nullopt
std::optional<int> even_value = is_even ? std::optional<int>(128) : std::nullopt;

// 可以用 std::optional 对象是否等于 std::nullopt 来判断 std::optional 对象是否有值
if (even_value != std::nullopt) {
    // 采用.value 获取 std::optional 对象的值
    std::cout << "has value, which is " << even_value.value() << std::endl;
} else {
    std::cout << "no value" << std::endl;
}
```

其实std::optional的作用和Python里面的`None`比较像，例如上面的例子用Python来写就是这样：
```python
is_even = True
even_value = 128 if is_even else None
if even_value is not None:
    print("has value, which is", even_value)
else:
    print("no value")
```
<!--more-->

### 2. 为什么要引入 std::optional
我觉得提出std::optional就是因为C++底层缺少`None` 这个表示，所以将std::nullopt和某种特定类型的变量合并在一起构造成一个`std::optional`对象，用以解决因为缺少之前`None`因而存在的一些不怎么直接的用法。

这里举个例子来说明前面提到的"不直接"的用法。这是一个寻找数组中的第一个非0元素的函数：
```cpp
int findFirstNonZero(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        if (arr[i] != 0) {
            return arr[i];
        }
    }
    return -1; // 如果数组中没有非0元素，则返回-1
}
```

可以看到，没找到元素时返回-1，所以当拿到-1时，没法判断是第一个非0元素为-1还是没找到非0元素。
改进方案是返回一个pair，第一个位置表示是否包含非0元素，第二个位置表示非0元素的值：
```cpp
#include <utility>

std::pair<bool, int> findFirstNonZero(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        if (arr[i] != 0) {
            return std::make_pair(true, arr[i]);
        }
    }
    return std::make_pair(false, -1); // 如果数组中没有非0元素，则返回false和-1
}
```

但这样其实比较繁琐且不直观，两个变量的解析和使用成本还是有些高，如果能用一个变量来完成的话就更简洁了。

采用std::optional可以简化上面的代码：
```cpp
#include <optional>

std::optional<int> findFirstNonZero(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        if (arr[i] != 0) {
            return arr[i];
        }
    }
    return std::nullopt; // 如果数组中没有非0元素，则返回std::nullopt
}
```
注意这里int类型的返回值可以隐式地转换为 std::optional 对象。

使用这个函数时也只需要判断一下返回值是否为`std::nullopt` 就可以。

总之可以将std::optional对象当作支持判断是否为NULL的对象的封装，在不确定对象是否存在的情况下，建议使用。

### 3. std::optional 的构造
空的 std::optional 对象可以用`std::nullopt` 或者`{}` 来构造，然后用`emplace` 函数来插入数值：
```cpp
    // 1.0 采用 std::nullopt 初始化再调用 emplace 插入值
    std::optional<int> val0 = std::nullopt;
    val0.emplace(128);
    std::cout << val0.value() << std::endl;

    // 1.1 采用 {} 初始化再调用 emplace 插入值
    std::optional<int> val1 = {};
    val1.emplace(128);
    std::cout << val1.value() << std::endl;
```
每次调用`emplace` 时，会清除掉之前的值，因此可以多次调用，且能保证每次都是最新的数值。

也可以用 `std::make_optional` 函数来构造：
```cpp
    // 1.7 采用 std::make_optional<T>(val) 初始化
    std::optional<int> val7 = std::make_optional<int>(128);
    std::cout << val7.value() << std::endl;

    // 1.8 采用 std::make_optional(val) 初始化，自动推导变量类型
    std::optional<int> val8 = std::make_optional(128);
    std::cout << val8.value() << std::endl;

```

除此之外还有很多种初始化 std::optional 对象的方法，都写在这个示例代码里面了，记得看注释：
```cpp
    // 1.2 采用 std::optional<T>(val) 初始化
    std::optional<int> val2 = std::optional<int>(128);
    std::cout << val2.value() << std::endl;

    // 1.3 采用 std::optional(val) 初始化，自动推导变量类型
    std::optional<int> val3 = std::optional(128);
    std::cout << val3.value() << std::endl;

    // 1.4 采用 std::optional<T>{val} 初始化
    std::optional<int> val4 = std::optional<int>{128};
    std::cout << val4.value() << std::endl;

    // 1.5 采用 std::optional{val} 初始化
    std::optional<int> val5 = std::optional{128};
    std::cout << val5.value() << std::endl;

    // 1.6 采用 {val} 初始化
    std::optional<int> val6 = {128};
    std::cout << val6.value() << std::endl;

```

### 4. std::optional 判断是否有值
判断 std::optional 对象是否有值可以用 `has_value`函数，或者判断是否不等于`std::nullopt`，或者直接用if语句对对象进行判断：
```cpp
    std::optional<int> result1 = find_the_first_postive_value(pos_values);

    if (result1.has_value()) {
        std::cout << result1.value() << std::endl;
    }
    
    if (result1 != std::nullopt) {
        std::cout << result1.value() << std::endl;
    }
    
    if (result1) {
        std::cout << result1.value() << std::endl;
    }
```

### 5. std::optional 获取值

获取值的话可以用`.value()` 函数，或者`*` 运算符：
```cpp
   if (result1) {
        std::cout << result1.value() << std::endl;
    }
    if (result1) {
        std::cout << *result1 << std::endl;
    }
```

如果想在std::optional对象为`std::nullopt`的情况下设置默认值的话，可以用`value_or` 函数：
```cpp
    std::optional<int> val9 = std::nullopt;
    std::cout << val9.value_or(-1) << std::endl; // 输出 -1
    val9.emplace(128);
    std::cout << val9.value_or(-1) << std::endl; // 输出 128
```

很明显，`value_or`函数中的默认值需要和optional对象的类型一致，否则会编译报错。

### 6. 没有值时的异常处理

如果在没有值的情况下调用`.value` 函数，会在运行时报错`std::bad_optional_access`:
```cpp
    std::optional<int> val10 = std::nullopt;
    std::cout << val10.value() << std::endl;
```

输出：
```plain
libc++abi: terminating due to uncaught exception of type std::bad_optional_access: bad_optional_access
```
所以建议使用`.value_or`来处理，如果要强行使用`.value`的话，需要使用 try-catch 语句：
```cpp
    std::optional<int> val11 = std::nullopt;
    try {
        std::cout << val11.value() << std::endl;
    } catch (const std::bad_optional_access& e) {
        std::cout << "==> error: " << e.what() << std::endl;
    }
```

### 7. 示例代码
上面的所有示例代码汇总：
```cpp
#include <iostream>
#include <optional>
#include <vector>

std::optional<int> find_the_first_postive_value(const std::vector<int>& values) {
    for (auto& val : values) {
        if (val > 0) {
            return std::optional<int>(val);
        }
    }

    return std::nullopt;
}

std::optional<int> find_the_first_postive_value_v2(const std::vector<int>& values) {
    auto it = std::find_if(values.begin(), values.end(), [](int val) { return val > 0; });
    return it != values.end() ? std::make_optional(*it) : std::nullopt;
}

void show_backend(std::optional<std::string> backend) {
    if (backend) {
        std::cout << "==> use set backend: " << backend.value() << std::endl;
    } else {
        std::cout << "==> use default backend: CPU" << std::endl;
    }
}

int main() {
    // std::optional 简单例子
    bool is_even = true;
    std::optional<int> even_value = is_even ? std::optional<int>(128) : std::nullopt;
    if (even_value != std::nullopt) {
        std::cout << "has value, which is " << even_value.value() << std::endl;
    } else {
        std::cout << "no value" << std::endl;
    }

    // 1. std::optional 对象的构造
    // 1.0 采用 std::nullopt 初始化再调用 emplace 插入值
    std::optional<int> val0 = std::nullopt;
    val0.emplace(128);
    val0.emplace(129);
    std::cout << val0.value() << std::endl;

    // 1.1 采用 {} 初始化再调用 emplace 插入值
    std::optional<int> val1 = {};
    val1.emplace(128);
    std::cout << val1.value() << std::endl;

    // 1.2 采用 std::optional<T>(val) 初始化
    std::optional<int> val2 = std::optional<int>(128);
    std::cout << val2.value() << std::endl;

    // 1.3 采用 std::optional(val) 初始化，自动推导变量类型
    std::optional<int> val3 = std::optional(128);
    std::cout << val3.value() << std::endl;

    // 1.4 采用 std::optional<T>{val} 初始化
    std::optional<int> val4 = std::optional<int>{128};
    std::cout << val4.value() << std::endl;

    // 1.5 采用 std::optional{val} 初始化
    std::optional<int> val5 = std::optional{128};
    std::cout << val5.value() << std::endl;

    // 1.6 采用 {val} 初始化
    std::optional<int> val6 = {128};
    std::cout << val6.value() << std::endl;

    // 1.7 采用 std::make_optional<T>(val) 初始化
    std::optional<int> val7 = std::make_optional<int>(128);
    std::cout << val7.value() << std::endl;

    // 1.8 采用 std::make_optional(val) 初始化，自动推导变量类型
    std::optional<int> val8 = std::make_optional(128);
    std::cout << val8.value() << std::endl;

    std::optional<int> val9 = std::nullopt;
    std::cout << val9.value_or(-1) << std::endl;
    val9.emplace(128);
    std::cout << val9.value_or(-1) << std::endl;

    //    std::optional<int> val10 = std::nullopt;
    //    std::cout << val10.value() << std::endl;

    std::optional<int> val11 = std::nullopt;
    try {
        std::cout << val11.value() << std::endl;
    } catch (const std::bad_optional_access& e) {
        std::cout << "==> error: " << e.what() << std::endl;
    }

    // 函数调用例子
    std::vector<int> neg_values = {-1, -3, -5};
    std::vector<int> pos_values = {1, 3, 5};

    auto result1 = find_the_first_postive_value_v2(pos_values);
    if (result1.has_value()) {
        std::cout << result1.value() << std::endl;
    }
    if (result1 != std::nullopt) {
        std::cout << result1.value() << std::endl;
    }
    if (result1) {
        std::cout << result1.value() << std::endl;
    }
    if (result1) {
        std::cout << *result1 << std::endl;
    }

    // try-catch 示例
    try {
        std::cout << result1.value() << std::endl;
    } catch (const std::bad_optional_access& e) {
        std::cout << "==> error: " << e.what() << std::endl;
    }

    show_backend(std::nullopt);
    show_backend(std::make_optional("CUDA"));

    auto my_backend = std::optional<std::string>{"MPS"};
    show_backend(my_backend);
    my_backend.emplace("DSP");
    show_backend(my_backend);

    std::optional<std::vector<int>> res = std::optional<std::vector<int>>({1, 2, 3});
    std::cout << res.value()[0] << std::endl;
}
```

可以通过`g++ -std=c++17 main.cpp  && ./a.out` 来编译运行。


### 8. 参考
1. https://en.cppreference.com/w/cpp/utility/optional
2. https://devblogs.microsoft.com/cppblog/stdoptional-how-when-and-why

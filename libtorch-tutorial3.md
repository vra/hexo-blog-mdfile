---
title: libtorch系列教程3：优雅地训练MNIST分类模型
date: 2023-06-30 22:38:16
tags:
- Libtorch
- Pytorch
- C++
- Python
---
在这篇文章中，我们对如何使用Libtorch进行MNIST分类模型的训练和测试进行详细描述。首先会浏览官方MNIST示例，然后对其进行模块化重构，为后续别的模型的训练提供 codebase。

由于Libtorch中包含很多和Pytorch中没有的类型，所以看Libtorch代码的时候时常会遇到不了解的函数或者类，这时候可以在[这里](https://github.com/pytorch/pytorch/tree/main/torch/csrc/api/include/torch)查找对应的类的实现，了解其作用。Libtorch C++ 代码中的注释虽然不多但基本够用了。

这里列举一些常见的类的代码路径，方便查询：
+ Datasets: https://github.com/pytorch/pytorch/blob/main/torch/csrc/api/include/torch/data/datasets/base.h
+ DataLoader:https://github.com/pytorch/pytorch/tree/main/torch/csrc/api/include/torch/data/dataloader/base.h
+ MNIST: https://github.com/pytorch/pytorch/blob/main/torch/csrc/api/include/torch/data/datasets/mnist.h
+ Stack: https://github.com/pytorch/pytorch/blob/main/torch/csrc/api/include/torch/data/transforms/stack.h
+ RandomSampler:  https://github.com/pytorch/pytorch/tree/main/torch/csrc/api/src/data/samplers/random.cpp
+ SequentialSampler: https://github.com/pytorch/pytorch/tree/main/torch/csrc/api/src/data/samplers/sequential.cpp

<!--more-->
+ 
### 1. 官方MNIST示例
Libtorch官方的训练代码仓库在[这里](https://github.com/pytorch/examples/tree/main/cpp)，拿里面的训练MNIST为例，代码如下：
```cpp
#include <torch/torch.h>

#include <cstddef>
#include <cstdio>
#include <iostream>
#include <string>
#include <vector>

// Where to find the MNIST dataset.
const char* kDataRoot = "./data";

// The batch size for training.
const int64_t kTrainBatchSize = 64;

// The batch size for testing.
const int64_t kTestBatchSize = 1000;

// The number of epochs to train.
const int64_t kNumberOfEpochs = 10;

// After how many batches to log a new update with the loss value.
const int64_t kLogInterval = 10;

struct Net : torch::nn::Module {
  Net()
      : conv1(torch::nn::Conv2dOptions(1, 10, /*kernel_size=*/5)),
        conv2(torch::nn::Conv2dOptions(10, 20, /*kernel_size=*/5)),
        fc1(320, 50),
        fc2(50, 10) {
    register_module("conv1", conv1);
    register_module("conv2", conv2);
    register_module("conv2_drop", conv2_drop);
    register_module("fc1", fc1);
    register_module("fc2", fc2);
  }

  torch::Tensor forward(torch::Tensor x) {
    x = torch::relu(torch::max_pool2d(conv1->forward(x), 2));
    x = torch::relu(
        torch::max_pool2d(conv2_drop->forward(conv2->forward(x)), 2));
    x = x.view({-1, 320});
    x = torch::relu(fc1->forward(x));
    x = torch::dropout(x, /*p=*/0.5, /*training=*/is_training());
    x = fc2->forward(x);
    return torch::log_softmax(x, /*dim=*/1);
  }

  torch::nn::Conv2d conv1;
  torch::nn::Conv2d conv2;
  torch::nn::Dropout2d conv2_drop;
  torch::nn::Linear fc1;
  torch::nn::Linear fc2;
};

template <typename DataLoader>
void train(
    size_t epoch,
    Net& model,
    torch::Device device,
    DataLoader& data_loader,
    torch::optim::Optimizer& optimizer,
    size_t dataset_size) {
  model.train();
  size_t batch_idx = 0;
  for (auto& batch : data_loader) {
    auto data = batch.data.to(device), targets = batch.target.to(device);
    optimizer.zero_grad();
    auto output = model.forward(data);
    auto loss = torch::nll_loss(output, targets);
    AT_ASSERT(!std::isnan(loss.template item<float>()));
    loss.backward();
    optimizer.step();

    if (batch_idx++ % kLogInterval == 0) {
      std::printf(
          "\rTrain Epoch: %ld [%5ld/%5ld] Loss: %.4f",
          epoch,
          batch_idx * batch.data.size(0),
          dataset_size,
          loss.template item<float>());
    }
  }
}

template <typename DataLoader>
void test(
    Net& model,
    torch::Device device,
    DataLoader& data_loader,
    size_t dataset_size) {
  torch::NoGradGuard no_grad;
  model.eval();
  double test_loss = 0;
  int32_t correct = 0;
  for (const auto& batch : data_loader) {
    auto data = batch.data.to(device), targets = batch.target.to(device);
    auto output = model.forward(data);
    test_loss += torch::nll_loss(
                     output,
                     targets,
                     /*weight=*/{},
                     torch::Reduction::Sum)
                     .template item<float>();
    auto pred = output.argmax(1);
    correct += pred.eq(targets).sum().template item<int64_t>();
  }

  test_loss /= dataset_size;
  std::printf(
      "\nTest set: Average loss: %.4f | Accuracy: %.3f\n",
      test_loss,
      static_cast<double>(correct) / dataset_size);
}

auto main() -> int {
  torch::manual_seed(1);

  torch::DeviceType device_type;
  if (torch::cuda::is_available()) {
    std::cout << "CUDA available! Training on GPU." << std::endl;
    device_type = torch::kCUDA;
  } else {
    std::cout << "Training on CPU." << std::endl;
    device_type = torch::kCPU;
  }
  torch::Device device(device_type);

  Net model;
  model.to(device);

  auto train_dataset = torch::data::datasets::MNIST(kDataRoot)
                           .map(torch::data::transforms::Normalize<>(0.1307, 0.3081))
                           .map(torch::data::transforms::Stack<>());
  const size_t train_dataset_size = train_dataset.size().value();
  auto train_loader =
      torch::data::make_data_loader<torch::data::samplers::SequentialSampler>(
          std::move(train_dataset), kTrainBatchSize);

  auto test_dataset = torch::data::datasets::MNIST(
                          kDataRoot, torch::data::datasets::MNIST::Mode::kTest)
                          .map(torch::data::transforms::Normalize<>(0.1307, 0.3081))
                          .map(torch::data::transforms::Stack<>());
  const size_t test_dataset_size = test_dataset.size().value();
  auto test_loader =
      torch::data::make_data_loader(std::move(test_dataset), kTestBatchSize);

  torch::optim::SGD optimizer(
      model.parameters(), torch::optim::SGDOptions(0.01).momentum(0.5));

  for (size_t epoch = 1; epoch <= kNumberOfEpochs; ++epoch) {
    train(epoch, model, device, *train_loader, optimizer, train_dataset_size);
    test(model, device, *test_loader, test_dataset_size);
  }
}
```

代码具体细节可以先不用理解，后文有一些说明。可以看到所有的模型搭建、数据读取、网络训练和测试代码都混在一个文件里面，别的几个例子里面也是类似的写法。

这样写当然是可以的，但对于习惯了Pytorch训练的我们来说，这样所有的代码在一个文件中的写法很不易读，
修改数据和网络都相互有影响，且不利用真正严肃地模型训练迭代。


### 2. 重构 MNIST 示例代码
所以一个简单的想法是改进写法，将DataLoader, Model 和训练逻辑拆分出来，分别进行模块化，放到单独的文件中处理。

#### 2.1 简单拆分的问题
第一次尝试是将Dataset和DataLoader放到一个模块中，网络定义放到一个模块中，训练和测试代码放到一个模块中。
但这样拆分遇到很大问题，核心原因是 Libtorch 的DataLoader类别太复杂了，对于我这种C++了解不深入的人来说改造难度太大。

举个例子，我们对MNIST Dataset类进行Normalize后Stack，然后构造一个DataLoader对象`train_loader`，代码如下：
```cpp
auto train_dataset = torch::data::datasets::MNIST(data_root)
                             .map(torch::data::transforms::Normalize<>(0.1307, 0.3081))
                             .map(torch::data::transforms::Stack<>());
auto train_loader =
        torch::data::make_data_loader<torch::data::samplers::SequentialSampler>(std::move(train_dataset), 64);
```
生成的`train_loader`对象的类型是：
```cpp
torch::disable_if_t<MapDataset<MapDataset<MNIST, Normalize<>>, Stack<>>::is_stateful || !std::is_constructible<SequentialSampler, size_t>::value, std::unique_ptr<StatelessDataLoader<MapDataset<MapDataset<MNIST, Normalize<>>, Stack<>>, SequentialSampler>>>
```
这个类型太复杂了……

因为官方示例是所有代码在一个文件，因此可以通过`auto` 来让编译器自动判定类型，省去了写着一长串类型的问题。

但如果我们要拆分DataLoader到单独的类里面的话，就没法使用`auto`，需要显式的指出DataLoader的类型，然而即使是这样一长串的类型写上了，还是会有不知道是哪里的问题，导致编译报错。

当然也有可能有简单的方法来解决这个问题，欢迎C++高手讨论指导。

这次体验让我真正体会到了动态类型语言的简洁性，以及Python的所有类型转C++会存在哪些坑。

#### 2.2 一种比较简单的重构方案
最后给出了一个妥协的方案：DataSet在单独的类中定义里面，而DataLoader在训练逻辑中构造，避免繁琐的类型问题。

整体代码结构如下：
```bash
├── CMakeLists.txt # CMake配置文件
├── main.cpp # 主入口
├── my_dataset.cpp # 数据集实现
├── my_dataset.h 
├── my_model.cpp # 模型定义
├── my_model.h
├── my_trainer.cpp # 训练和测试脚手架代码
└── my_trainer.h
```

##### 2.2.1 CMake 配置文件
CMake 配置文件`CMakeLists.txt`中将几个实现文件加入到编译依赖即可，别的部分与前两篇文章中的类似。
```cmake
cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
project(mnist_train)

# 需要找到Libtorch
find_package(Torch REQUIRED)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")

add_executable(${PROJECT_NAME} main.cpp my_model.cpp my_dataset.cpp my_trainer.cpp)
target_link_libraries(${PROJECT_NAME} "${TORCH_LIBRARIES}")

# Libtorch是基于C++14来实现的
set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 14)

```

##### 2.2.2 主入口文件定义
主入口文件实现了超参数设置，网络和数据集初始化，以及调用Trainer进行训练和测试：
```cpp
#include <string>

#include <torch/torch.h>

#include "my_dataset.h"
#include "my_model.h"
#include "my_trainer.h"

int main() {
  // 超参数设置
  std::string data_root = "./data";
  int train_batch_size = 128;
  int test_batch_size = 1000;
  int total_epoch_num = 30;
  int log_interval = 10;
  int num_workers = 32;

  // 设置随机数种子
  torch::manual_seed(1);

  // 获取设备类型
  torch::DeviceType device_type = torch::kCPU;
  if (torch::cuda::is_available()) {
    device_type = torch::kCUDA;
  }
  torch::Device device(device_type);

  // 构造网络
  MyModel model;
  model.to(device);

  // 设置优化器
  torch::optim::SGD optimizer(
      model.parameters(), torch::optim::SGDOptions(0.01).momentum(0.5));

  // 构造训练和测试dataset
  auto train_dataset =
      MyDataset(data_root, torch::data::datasets::MNIST::Mode::kTrain);
  auto test_dataset =
      MyDataset(data_root, torch::data::datasets::MNIST::Mode::kTest);

  // Trainer初始化
  auto trainer = MyTrainer(log_interval);
  for (size_t epoch = 1; epoch < total_epoch_num; ++epoch) {
   // 运行训练
    trainer.train(
        epoch,
        model,
        optimizer,
        device,
        train_dataset,
        train_batch_size,
        num_workers);
        
    // 运行测试
    trainer.test(model, device, test_dataset, test_batch_size, num_workers);
  }
}
```
##### 2.2.3 网络定义
网络结构采用简单的LeNet，两个conv层和2个fc层。
头文件 my_model.h 内容:
```cpp
#pragma once
#include <torch/torch.h>

class MyModel : public torch::nn::Module {
 public:
  MyModel();
  torch::Tensor forward(torch::Tensor x);

 private:
  torch::nn::Conv2d conv1 = nullptr;
  torch::nn::Conv2d conv2 = nullptr;
  torch::nn::Dropout2d conv2_drop;
  torch::nn::Linear fc1 = nullptr;
  torch::nn::Linear fc2 = nullptr;
};
```

实现文件 my_model.cpp:
```cpp
#include "my_model.h"

MyModel::MyModel() {
  conv1 = torch::nn::Conv2d(torch::nn::Conv2dOptions(1, 10, 5));
  conv2 = torch::nn::Conv2d(torch::nn::Conv2dOptions(10, 20, 5));
  fc1 = torch::nn::Linear(320, 50);
  fc2 = torch::nn::Linear(50, 10);

  register_module("conv1", conv1);
  register_module("conv2", conv2);
  register_module("conv2_drop", conv2_drop);
  register_module("fc1", fc1);
  register_module("fc2", fc2);
}

torch::Tensor MyModel::forward(torch::Tensor x) {
  // conv1
  x = conv1->forward(x);
  x = torch::max_pool2d(x, 2);
  x = torch::relu(x);

  // conv2
  x = conv2->forward(x);
  x = conv2_drop->forward(x);
  x = torch::max_pool2d(x, 2);
  x = torch::relu(x);

  // fc1
  x = x.view({-1, 320});
  x = fc1->forward(x);
  x = torch::relu(x);

  // dropout
  x = torch::dropout(x, 0.5, is_training());

  // fc2
  x = fc2->forward(x);

  // log softmax
  x = torch::log_softmax(x, 1);

  return x;
}
```
可以看到网络的定义还是比较简单直接，可以直接从Python 网络定义迁移过去，几个核心点：
+ 网络类的定义需要继承`torch::nn::Module` 类
+ 实现`forward` 函数来进行网络前项运算，其中每个层需要显式地调用`forward` 函数


##### 2.2.4 数据集定义
由于 Libtorch 自带 MNIST的实现，我们这里只是做了一个简单的封装，作为模块化的例子。
头文件`my_dataset.h` 内容：
```cpp
#pragma once
#include <torch/torch.h>

class MyDataset {
 public:
  MyDataset(
      const std::string& data_root,
      torch::data::datasets::MNIST::Mode phase);

 public:
  torch::data::datasets::MNIST mnist_dataset;
};
```

实现文件`my_dataset.cpp` 内容：
```cpp
#include "my_dataset.h"

MyDataset::MyDataset(
    const std::string& data_root,
    torch::data::datasets::MNIST::Mode phase)
    : mnist_dataset(torch::data::datasets::MNIST(data_root, phase)) {}
```
这里有一个需要注意的点，由于MNIST类本身没有默认构造函数，所以在`MyDataset` 类的初始化列表中就必须给成员变量`mnist_dataset`赋值，否则会报下面的错:
```plain
constructor for 'MyDataset' must explicitly initialize the member 'mnist_dataset' which does not have a default constructor
```


##### 2.2.5 Trainer定义
Trainer 包含训练和测试的两个函数，对数据和网络，优化器等输入进行计算，得到输出，计算loss和准确率。
头文件`my_trainer.h`内容：
```cpp
#pragma once
#include <torch/torch.h>

#include "my_dataset.h"
#include "my_model.h"

class MyTrainer {
 public:
  MyTrainer(int log_interval) : log_interval_(log_interval){};

  void train(
      size_t epoch,
      MyModel& model,
      torch::optim::Optimizer& optimizer,
      torch::Device device,
      MyDataset& train_dataset,
      int batch_size,
      int num_workers);

  void test(
      MyModel& model,
      torch::Device device,
      MyDataset& test_dataset,
      int batch_size,
      int num_workers);

 private:
  int log_interval_;
};
```

实现文件`my_trainer.cpp` 内容：
```cpp
#include "my_trainer.h"

#include <torch/torch.h>

#include <cstdio>
#include <string>
#include <vector>

void MyTrainer::train(
    size_t epoch,
    MyModel& model,
    torch::optim::Optimizer& optimizer,
    torch::Device device,
    MyDataset& train_dataset,
    int batch_size,
    int num_workers) {
  model.train();

  // 对MNIST数据进行Normalize和Stack（将多个Tensor stack成一个Tensor)
  auto dataset = train_dataset.mnist_dataset
                     .map(torch::data::transforms::Normalize<>(0.1307, 0.3081))
                     .map(torch::data::transforms::Stack<>());

  // 构造 DataLoader, 设置 batch size 和 worker 数目
  auto data_loader = torch::data::make_data_loader(
      dataset,
      torch::data::DataLoaderOptions()
          .batch_size(batch_size)
          .workers(num_workers));
  auto dataset_size = dataset.size().value();

  size_t batch_idx = 0;
  // 网络训练
  for (auto& batch : *data_loader) {
    // 获取数据和label
    auto data = batch.data.to(device);
    auto targets = batch.target.to(device);

    // 优化器 梯度清零
    optimizer.zero_grad();

    // 模型前向操作，得到预测输出
    auto output = model.forward(data);

    // 计算loss
    auto loss = torch::nll_loss(output, targets);

    // loss 反传
    loss.backward();
    optimizer.step();

    // 打印log信息
    if (batch_idx++ % log_interval_ == 0) {
      std::printf(
          "\rTrain Epoch: %ld [%5llu/%5ld] Loss: %.4f",
          epoch,
          batch_idx * batch.data.size(0),
          dataset_size,
          loss.template item<float>());
    }
  }
}

void MyTrainer::test(
    MyModel& model,
    torch::Device device,
    MyDataset& test_dataset,
    int batch_size,
    int num_workers) {
  // 测试时要将模型置为eval模式
  model.eval();
  double test_loss = 0;
  int32_t correct = 0;

  // 对MNIST数据进行Normalize和Stack（将多个Tensor stack成一个Tensor)
  auto dataset = test_dataset.mnist_dataset
                     .map(torch::data::transforms::Normalize<>(0.1307, 0.3081))
                     .map(torch::data::transforms::Stack<>());

  // 构造 DataLoader, 设置 batch size 和 worker 数目
  auto data_loader = torch::data::make_data_loader(
      dataset,
      torch::data::DataLoaderOptions()
          .batch_size(batch_size)
          .workers(num_workers));
  auto dataset_size = dataset.size().value();

  for (const auto& batch : *data_loader) {
    // 获取数据和label
    auto data = batch.data.to(device);
    auto targets = batch.target.to(device);

    // 模型前向操作，得到预测输出
    auto output = model.forward(data);

    // 计算测试时的 loss
    test_loss += torch::nll_loss(
                     output,
                     targets,
                     /*weight=*/{},
                     torch::Reduction::Sum)
                     .item<float>();
    auto pred = output.argmax(1);
    correct += pred.eq(targets).sum().template item<int64_t>();
  }

  test_loss /= dataset_size;
  std::printf(
      "\nTest set: Average loss: %.4f | Accuracy: %.3f\n",
      test_loss,
      static_cast<double>(correct) / dataset_size);
}
```

##### 2.2.6 编译和运行方式
我们基于CMake 编译上面的代码，同时下载MNIST数据集，完整的执行命令如下：
```bash
mkdir build
cd build
cmake ..  -DCMAKE_PREFIX_PATH=`python -c 'import torch;print(torch.utils.cmake_prefix_path)'`
make -j8
# 下载MNIST数据
mkdir data && cd data
wget "http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz" && gunzip train-images-idx3-ubyte.gz
wget "http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz" && gunzip train-labels-idx1-ubyte.gz
wget "http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz" && gunzip t10k-images-idx3-ubyte.gz
wget "http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz" && gunzip t10k-labels-idx1-ubyte.gz
cd ../

# 运行可执行文件
./mnist_train
```
训练和测试输出如下：
```plain
Train Epoch: 1 [59008/60000] Loss: 0.6824
Test set: Average loss: 0.3265 | Accuracy: 0.910
Train Epoch: 2 [59008/60000] Loss: 0.5521
Test set: Average loss: 0.2018 | Accuracy: 0.941
Train Epoch: 3 [59008/60000] Loss: 0.3403
Test set: Average loss: 0.1523 | Accuracy: 0.954
Train Epoch: 4 [59008/60000] Loss: 0.3885
Test set: Average loss: 0.1236 | Accuracy: 0.965
Train Epoch: 5 [59008/60000] Loss: 0.3502
Test set: Average loss: 0.1083 | Accuracy: 0.967
Train Epoch: 6 [59008/60000] Loss: 0.1389
Test set: Average loss: 0.0961 | Accuracy: 0.970
Train Epoch: 7 [59008/60000] Loss: 0.3550
Test set: Average loss: 0.0899 | Accuracy: 0.972
...
```

可以看到准确率在逐渐提升。

这篇文章的内容主要就是这些，后面会根据训练一个实际一些的例子，比如nanoGPT，将在本文的codebase基础上，主要覆盖下面的内容：
+ 自定义数据集的Dataset类的搭建
+ 复杂网络的定义(如ResNet, Transformer)
+ 模型checkpoint的保存和读取

欢迎点赞和关注！

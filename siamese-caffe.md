title: Caffe中的Siamese网络
date: 2016-12-13 21:05:01
tags:
 - Caffe
 - Siamese Network
 - 深度学习
---
Siamese原意是"泰国的，泰国人"，而与之相关的一个比较常见的词是"Siamese twin"， 意思是是"连体双胞胎"，所以Siamemse Network是从这个意思转变而来，指的是结构非常相似的两路网络，分别训练，**但共享各个层的参数**，在最后有一个连接的部分。Siamese网络对于相似性比较的场景比较有效。此外Siamese因为共享参数，所以能减少训练过程中的参数个数。[这里](http://vision.ia.ac.cn/zh/senimar/reports/Siamese-Network-Architecture-and-Applications-in-Computer-Vision.pdf)的slides讲解了Siamese网络在深度学习中的应用。这里我参照Caffe中的[Siamese文档](https://github.com/BVLC/caffe/tree/master/examples/siamese)， 以LeNet为例，简单地总结下Caffe中Siamese网络的prototxt文件的写法。 
<!--more-->

### 1. Data层
Data层输入可以是LMDB和LevelDB格式的数据，这种格式的数据可以通过参照`$CAFFE_ROOT/examples/siamese/create_mnist_siamese.sh`来生成（该脚本是从MNIST原先的格式生成DB文件，如果要从JPEG格式的图片来生成DB文件，需要进行一定的修改）。  
Data层有2个输出，一个是`pair_data`，表示配对好的图片对;另一个是`sim`，表示图片对是否属于同一个label。 

### 2. Slice层
Slice层是Caffe中的一个工具层，功能就是把输入的层(bottom)切分成几个输出层(top)。官网给出的如下例子:
```bash
layer {
  name: "slicer_label"
  type: "Slice"
  bottom: "label"
  ## Example of label with a shape N x 3 x 1 x 1
  top: "label1"
  top: "label2"
  top: "label3"
  slice_param {
    axis: 1
    slice_point: 1
    slice_point: 2
  }
}
```
完成的功能就是把slicer_label划分成3份。`axis`表示划分的维度，这里1表示在第二个维度上划分;`slice_point`表示划分的中间的点，分别是`1`，`2`表示在1-2层和2-3层之间进行一个划分。
在Siamese网络中，为了对数据对进行单独的训练，需要在Data层后面接一个Slice层，将数据均匀地划分为2个部分。 

### 3. 共享层
后面的卷积层，Pooling层，Relu层对于两路网络是没有区别的，所以可以直接写好一路后，复制一份在后面作为另一路，不过得将name，bottom和top的名字改成不一样的(示例中第二路的名字都是在第一路对应层的名字后面加了个`_p`表示pair)。 
### 4. 如何共享参数
两路网络如何共享参数呢？Caffe里是这样实现的:在每路中对应的层里面都定义一个同名的参数，这样更新参数的时候就可以共享参数了。如下面的例子:
```bash
...

layer {                                                                         
  name: "ip2"                                                                   
  type: "InnerProduct"                                                          
  bottom: "ip1"                                                                 
  top: "ip2"                                                                    
  param {                                                                       
    name: "ip2_w"                                                               
    lr_mult: 1                                                                  
  }                
}

...

layer {                                                                         
  name: "ip2_p"                                                                 
  type: "InnerProduct"                                                          
  bottom: "ip1_p"                                                               
  top: "ip2_p"                                                                  
  param {                                                                       
    name: "ip2_w"                                                               
    lr_mult: 1                                                                  
  }              
}

...

```
上面例子中，两路网络对应层都定义了`ip2_w`的参数，这样训练的时候就可以共享这个变量的值了。 

### 5. feature层
在2个全连接层后，我们将原来的分类的sofatmax层改为输出一个2维向量的全连接层:
```bash
layer {                                                                         
  name: "feat"                                                                  
  type: "InnerProduct"                                                          
  bottom: "ip2"                                                                 
  top: "feat"                                                                   
  param {                                                                       
    name: "feat_w"                                                              
    lr_mult: 1                                                                  
  }                                                                             
  param {                                                                       
    name: "feat_b"                                                              
    lr_mult: 2                                                                  
  }                                                                             
  inner_product_param {                                                         
    num_output: 2                                                               
    weight_filler {                                                             
      type: "xavier"                                                            
    }                                                                           
    bias_filler {                                                               
      type: "constant"                                                          
    }                                                                           
  }                                                                             
}                      
```

### 6. ContrastiveLoss层
在两个feature产生后，就可以利用2个feature和前面定义的`sim`来计算loss了。Siamese网络采用了一个叫做["ContrastiveLoss"](http://yann.lecun.com/exdb/publis/pdf/hadsell-chopra-lecun-06.pdf)的loss计算方式，如果两个图片越相似，则loss越小;如果越不相似，则loss越大。
```bash
layer {                                                                         
  name: "loss"                                                                  
  type: "ContrastiveLoss"                                                       
  bottom: "feat"                                                                
  bottom: "feat_p"                                                              
  bottom: "sim"                                                                 
  top: "loss"                                                                   
  contrastive_loss_param {                                                      
    margin: 1                                                                   
  }                                                                             
}                
```

### 7. 网络结构的可视化
上面就是所有的网络结构，利用`$CAFFE_ROOT/python/draw_net.py`这个脚本可以画出网络结构，如图所示:
![Lenet的Siamese网络结构](/imgs/prototxt.jpg)

整个网络的完整内容如下:
```bash
name: "mnist_siamese_train_test"
layer {
  name: "pair_data"
  type: "Data"
  top: "pair_data"
  top: "sim"
  include {
    phase: TRAIN
  }
  transform_param {
    scale: 0.00390625
  }
  data_param {
    source: "examples/siamese/mnist_siamese_train_leveldb"
    batch_size: 64
  }
}
layer {
  name: "pair_data"
  type: "Data"
  top: "pair_data"
  top: "sim"
  include {
    phase: TEST
  }
  transform_param {
    scale: 0.00390625
  }
  data_param {
    source: "examples/siamese/mnist_siamese_test_leveldb"
    batch_size: 100
  }
}
layer {
  name: "slice_pair"
  type: "Slice"
  bottom: "pair_data"
  top: "data"
  top: "data_p"
  slice_param {
    slice_dim: 1
    slice_point: 1
  }
}
layer {
  name: "conv1"
  type: "Convolution"
  bottom: "data"
  top: "conv1"
  param {
    name: "conv1_w"
    lr_mult: 1
  }
  param {
    name: "conv1_b"
    lr_mult: 2
  }
  convolution_param {
    num_output: 20
    kernel_size: 5
    stride: 1
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "pool1"
  type: "Pooling"
  bottom: "conv1"
  top: "pool1"
  pooling_param {
    pool: MAX
    kernel_size: 2
    stride: 2
  }
}
layer {
  name: "conv2"
  type: "Convolution"
  bottom: "pool1"
  top: "conv2"
  param {
    name: "conv2_w"
    lr_mult: 1
  }
  param {
    name: "conv2_b"
    lr_mult: 2
  }
  convolution_param {
    num_output: 50
    kernel_size: 5
    stride: 1
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "pool2"
  type: "Pooling"
  bottom: "conv2"
  top: "pool2"
  pooling_param {
    pool: MAX
    kernel_size: 2
    stride: 2
  }
}
layer {
  name: "ip1"
  type: "InnerProduct"
  bottom: "pool2"
  top: "ip1"
  param {
    name: "ip1_w"
    lr_mult: 1
  }
  param {
    name: "ip1_b"
    lr_mult: 2
  }
  inner_product_param {
    num_output: 500
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "relu1"
  type: "ReLU"
  bottom: "ip1"
  top: "ip1"
}
layer {
  name: "ip2"
  type: "InnerProduct"
  bottom: "ip1"
  top: "ip2"
  param {
    name: "ip2_w"
    lr_mult: 1
  }
  param {
    name: "ip2_b"
    lr_mult: 2
  }
  inner_product_param {
    num_output: 10
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "feat"
  type: "InnerProduct"
  bottom: "ip2"
  top: "feat"
  param {
    name: "feat_w"
    lr_mult: 1
  }
  param {
    name: "feat_b"
    lr_mult: 2
  }
  inner_product_param {
    num_output: 2
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "conv1_p"
  type: "Convolution"
  bottom: "data_p"
  top: "conv1_p"
  param {
    name: "conv1_w"
    lr_mult: 1
  }
  param {
    name: "conv1_b"
    lr_mult: 2
  }
  convolution_param {
    num_output: 20
    kernel_size: 5
    stride: 1
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "pool1_p"
  type: "Pooling"
  bottom: "conv1_p"
  top: "pool1_p"
  pooling_param {
    pool: MAX
    kernel_size: 2
    stride: 2
  }
}
layer {
  name: "conv2_p"
  type: "Convolution"
  bottom: "pool1_p"
  top: "conv2_p"
  param {
    name: "conv2_w"
    lr_mult: 1
  }
  param {
    name: "conv2_b"
    lr_mult: 2
  }
  convolution_param {
    num_output: 50
    kernel_size: 5
    stride: 1
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "pool2_p"
  type: "Pooling"
  bottom: "conv2_p"
  top: "pool2_p"
  pooling_param {
    pool: MAX
    kernel_size: 2
    stride: 2
  }
}
layer {
  name: "ip1_p"
  type: "InnerProduct"
  bottom: "pool2_p"
  top: "ip1_p"
  param {
    name: "ip1_w"
    lr_mult: 1
  }
  param {
    name: "ip1_b"
    lr_mult: 2
  }
  inner_product_param {
    num_output: 500
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "relu1_p"
  type: "ReLU"
  bottom: "ip1_p"
  top: "ip1_p"
}
layer {
  name: "ip2_p"
  type: "InnerProduct"
  bottom: "ip1_p"
  top: "ip2_p"
  param {
    name: "ip2_w"
    lr_mult: 1
  }
  param {
    name: "ip2_b"
    lr_mult: 2
  }
  inner_product_param {
    num_output: 10
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "feat_p"
  type: "InnerProduct"
  bottom: "ip2_p"
  top: "feat_p"
  param {
    name: "feat_w"
    lr_mult: 1
  }
  param {
    name: "feat_b"
    lr_mult: 2
  }
  inner_product_param {
    num_output: 2
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "loss"
  type: "ContrastiveLoss"
  bottom: "feat"
  bottom: "feat_p"
  bottom: "sim"
  top: "loss"
  contrastive_loss_param {
    margin: 1
  }
}
```

### 8. 训练过程
训练过程与别的网络是一样的，这里就不具体展开了。

### 9. 参考内容
 1. <https://www.quora.com/What-are-Siamese-neural-networks-what-applications-are-they-good-for-and-why>
 2. <http://vision.ia.ac.cn/zh/senimar/reports/Siamese-Network-Architecture-and-Applications-in-Computer-Vision.pdf>

title: 更新Faster-RCNN代码到最新版的caffe
date: 2017-11-15 00:00:28
tags:
 - Linux
 - Caffe
 - Deep Learning
 - Python
---


因为CuDNN函数接口更新的原因，以前用低版本写的项目在新版本的CuDNN环境下编译就会出问题。例如，[py-faster-rcnn](https://github.com/rbgirshick/py-faster-rcnn)代码在最新版的CuDNN6上面编译时就会报错。  
解决这个问题的一个方法是禁用CUDNN，即修改`Makefile.config`里面的第5行，在前面加`#`。这种方法没法使用CuDNN加速，不推荐。这里我们使用一种比较土的方法，即将使用了旧的CuDNN函数的文件都换成新的caffe里面的文件即可。  
<!--more-->

将所有要修改的文件和命令写在下面这个bash文件里，只要修改`CAFFE_ROOT` 和`CAFFE_FAST_RCNN`的值，然后调用这个bash文件就可以用了：
```bash
# set path of lastest caffe and caffe in py-faster-rcnn                                                                                                                                                                                                                        
CAFFE_ROOT=/data1/public/caffe                                                                                                                                                                                                                                                 
CAFFE_FAST_RCNN=/data6/yunfeng/py-faster-rcnn/caffe-fast-rcnn                                                                                                                                                                                                                  
                                                                                                                                                                                                                                                                               
# copy head files                                                                                                                                                                                                                                                              
cp $CAFFE_ROOT/include/caffe/util/cudnn.hpp $CAFFE_FAST_RCNN/include/caffe/util/cudnn.hpp                                                                                                                                                                                      
                                                                                                                                                                                                                                                                               
cp $CAFFE_ROOT/include/caffe/layers/cudnn_conv_layer.hpp $CAFFE_FAST_RCNN/include/caffe/layers/cudnn_conv_layer.hpp                                                                                                                                                            
cp $CAFFE_ROOT/include/caffe/layers/cudnn_lcn_layer.hpp $CAFFE_FAST_RCNN/include/caffe/layers/cudnn_lcn_layer.hpp                                                                                                                                                              
cp $CAFFE_ROOT/include/caffe/layers/cudnn_lrn_layer.hpp $CAFFE_FAST_RCNN/include/caffe/layers/cudnn_lrn_layer.hpp                                                                                                                                                              
cp $CAFFE_ROOT/include/caffe/layers/cudnn_pooling_layer.hpp $CAFFE_FAST_RCNN/include/caffe/layers/cudnn_pooling_layer.hpp                                                                                                                                                      
cp $CAFFE_ROOT/include/caffe/layers/cudnn_relu_layer.hpp $CAFFE_FAST_RCNN/include/caffe/layers/cudnn_relu_layer.hpp                                                                                                                                                            
cp $CAFFE_ROOT/include/caffe/layers/cudnn_sigmoid_layer.hpp $CAFFE_FAST_RCNN/include/caffe/layers/cudnn_sigmoid_layer.hpp                                                                                                                                                      
cp $CAFFE_ROOT/include/caffe/layers/cudnn_softmax_layer.hpp $CAFFE_FAST_RCNN/include/caffe/layers/cudnn_softmax_layer.hpp                                                                                                                                                      
cp $CAFFE_ROOT/include/caffe/layers/cudnn_tanh_layer.hpp $CAFFE_FAST_RCNN/include/caffe/layers/cudnn_tanh_layer.hpp                                                                                                                                                            
                                                                                                                                                                                                                                                                               
# copy cpp files                                                                                                                                                                                                                                                               
cp $CAFFE_ROOT/src/caffe/layers/cudnn_conv_layer.cpp $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_conv_layer.cpp                                                                                                                                                                    
cp $CAFFE_ROOT/src/caffe/layers/cudnn_lcn_layer.cpp $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_lcn_layer.cpp                                                                                                                                                                      
cp $CAFFE_ROOT/src/caffe/layers/cudnn_lrn_layer.cpp $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_lrn_layer.cpp                                                                                                                                                                      
cp $CAFFE_ROOT/src/caffe/layers/cudnn_pooling_layer.cpp $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_pooling_layer.cpp                                                                                                                                                              
cp $CAFFE_ROOT/src/caffe/layers/cudnn_relu_layer.cpp $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_relu_layer.cpp                                                                                                                                                                    
cp $CAFFE_ROOT/src/caffe/layers/cudnn_sigmoid_layer.cpp $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_sigmoid_layer.cpp                                                                                                                                                              
cp $CAFFE_ROOT/src/caffe/layers/cudnn_softmax_layer.cpp $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_softmax_layer.cpp                                                                                                                                                              
cp $CAFFE_ROOT/src/caffe/layers/cudnn_tanh_layer.cpp $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_tanh_layer.cpp                                                                                                                                                                    
                                                                                                                                                                                                                                                                               
# copy cu files                                                                                                                                                                                                                                                                
cp $CAFFE_ROOT/src/caffe/layers/cudnn_conv_layer.cu $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_conv_layer.cu                                                                                                                                                                      
cp $CAFFE_ROOT/src/caffe/layers/cudnn_lcn_layer.cu $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_lcn_layer.cu                                                                                                                                                                        
cp $CAFFE_ROOT/src/caffe/layers/cudnn_lrn_layer.cu $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_lrn_layer.cu                                                                                                                                                                        
cp $CAFFE_ROOT/src/caffe/layers/cudnn_pooling_layer.cu $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_pooling_layer.cu                                                                                                                                                                
cp $CAFFE_ROOT/src/caffe/layers/cudnn_relu_layer.cu $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_relu_layer.cu                                                                                                                                                                      
cp $CAFFE_ROOT/src/caffe/layers/cudnn_sigmoid_layer.cu $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_sigmoid_layer.cu                                                                                                                                                                
cp $CAFFE_ROOT/src/caffe/layers/cudnn_softmax_layer.cu $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_softmaxlayer.cu                                                                                                                                                                 
cp $CAFFE_ROOT/src/caffe/layers/cudnn_tanh_layer.cu $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_tanh_layer.cu                                                                                                                                                                      
                                                                                                                                                                                                                                                                               
                                                                                                                                                                                                                                                                               
# update source code using v3 of cudnn                                                                                                                                                                                                                                         
sed -i 's/cudnnConvolutionBackwardData_v3/cudnnConvolutionBackwardData/g' $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_conv_layer.cu                                                                                                                                                
sed -i 's/cudnnConvolutionBackwardFilter_v3/cudnnConvolutionBackwardFilter/g' $CAFFE_FAST_RCNN/src/caffe/layers/cudnn_conv_layer.cu    
```
最后的两行是修改`src/caffe/layers/cudnn_conv_layer.cu`,将其中的`cudnnConvolutionBackwardData_v3` 替换为`cudnnConvolutionBackwardData`，将`cudnnConvolutionBackwardFilter_v3`替换为`cudnnConvolutionBackwardFilter`。   
我已经将上述的脚本放到了GitHub上，可以从[这里](https://github.com/vra/update-cudnn-of-py-faster-rcnn)下载，下载后修改`CAFFE_ROOT` 和`CAFFE_FAST_RCNN`的路径，就可以直接运行脚本，修改文件了。  

然后重新make, make pycaffe 即可。  
参考：
<http://www.cnblogs.com/klitech/p/7651825.html>

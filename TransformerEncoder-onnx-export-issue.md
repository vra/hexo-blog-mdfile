---
title: TransformerEncoder导出onnx问题解决
date: 2025-01-29 16:38:07
tags:
- Pytorch
- Python
- Transformer
- ONNX
---
### 1. 问题说明
在使用Pytorch的TransformerEncoder时，导出onnx会将时序长度固定，导致没法采用变长输入，例如下面的简单例子复现了这个问题：
```python
import torch
import torch.nn as nn


class SimpleTransformer(nn.Module):
    def __init__(self, input_dim=512, num_layers=6, nhead=8):
        super().__init__()
        # 创建Transformer编码器层
        encoder_layer = nn.TransformerEncoderLayer(
            d_model=input_dim,
            nhead=nhead,
            dim_feedforward=2048,
            dropout=0.1,
            activation="relu",
            batch_first=True,  # 使用batch_first格式
        )

        # 创建Transformer编码器
        self.transformer_encoder = nn.TransformerEncoder(
            encoder_layer, num_layers=num_layers
        )

    def forward(self, x):
        # 输入形状: (batch_size, seq_len, input_dim)
        x = self.input_proj(x)
        output = self.transformer_encoder(x)
        return output


# 实例化模型
model = SimpleTransformer(input_dim=512, num_layers=2, nhead=8)
model.eval()  # 设置为评估模式

# 创建示例输入（batch_size=2, seq_len=10, input_dim=512）
dummy_input = torch.randn(2, 10, 512)

# 导出ONNX模型
torch.onnx.export(
    model,
    (dummy_input,),
    "transformer_encoder.onnx",
    do_constant_folding=True,  # 优化常量折叠
    input_names=["input"],  # 输入节点名称
    output_names=["output"],  # 输出节点名称
    dynamo=True,
)

print("ONNX model exported successfully!")

# 验证导出的模型
import onnxruntime as ort
import numpy as np

dummy_input2 = torch.randn(2, 11, 512)
ort_session = ort.InferenceSession("transformer_encoder.onnx")
outputs = ort_session.run(
    None,
    {"input": dummy_input2.numpy()}
)
print("ONNX output shape:", outputs[0].shape)
```
导出onnx时采用的时序长度是10，验证时采用时序长度11，运行时会报错：
```bash
2025-01-29 14:17:25.266794 [E:onnxruntime:, sequential_executor.cc:516 ExecuteKernel] Non-zero status code returned while running Reshape node. Name:'/transformer_encoder/layers.0/self_attn/Reshape_4' Status Message: /Users/runner/work/1/s/onnxruntime/core/providers/cpu/tensor/reshape_helper.h:47 onnxruntime::ReshapeHelper::ReshapeHelper(const onnxruntime::TensorShape &, onnxruntime::TensorShapeVector &, bool) input_shape_size == size was false. The input tensor cannot be reshaped to the requested shape. Input shape:{11,2,512}, requested shape:{10,16,64}

Traceback (most recent call last):
  File "/Users/ws/export.py", line 63, in <module>
    outputs = ort_session.run(
              ^^^^^^^^^^^^^^^^
  File "/Users/ws/miniforge3/lib/python3.12/site-packages/onnxruntime/capi/onnxruntime_inference_collection.py", line 266, in run
    return self._sess.run(output_names, input_feed, run_options)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
onnxruntime.capi.onnxruntime_pybind11_state.RuntimeException: [ONNXRuntimeError] : 6 : RUNTIME_EXCEPTION : Non-zero status code returned while running Reshape node. Name:'/transformer_encoder/layers.0/self_attn/Reshape_4' Status Message: /Users/runner/work/1/s/onnxruntime/core/providers/cpu/tensor/reshape_helper.h:47 onnxruntime::ReshapeHelper::ReshapeHelper(const onnxruntime::TensorShape &, onnxruntime::TensorShapeVector &, bool) input_shape_size == size was false. The input tensor cannot be reshaped to the requested shape. Input shape:{11,2,512}, requested shape:{10,16,64}
```

尝试了Pytorch 2+ 提供的TorchDynamo-based ONNX Exporter（torch.onnx.export增加`dynamo=True`参数），也是同样的报错。

### 2. 如何解决
这个问题在Pytorch的GitHub 上有几个issue都在讨论，并且也给出了解决方案，不过不知道为什么官方一直没有集成修复代码。

修复方式也比较简单，修改`torch/nn.functional.py`中的两行代码即可。具体操作如下。

首先定位到当前python环境的functional.py的路径，采用下面的一行命令即可：
```bash
python -c "import torch, os; print(os.path.join(os.path.dirname(torch.__file__), 'nn', 'functional.py'))"
```
然后打开这个文件，搜索`k = k.view(k.shape[0`，只有一处匹配，大概在6200行，内容是：
```python
k = k.view(k.shape[0], bsz * num_heads, head_dim).transpose(0, 1)
```
可用看到这里调用了k.shape[0]，在导出onnx时被固定了。将这一句修改为
```python
k = k.view(-1, bsz * num_heads, head_dim).transpose(0, 1)
```

同样的，搜索`v = v.view(v.shape[0]`，也只有一处匹配，紧接着上面的代码，原始内容：
```python
v = v.view(v.shape[0], bsz * num_heads, head_dim).transpose(0, 1)
```
修改为
```python
v = v.view(-1, bsz * num_heads, head_dim).transpose(0, 1)
```

保存文件，再运行上面导出和验证onnx的脚本，一切正常了。

这种方式需要修改Pytorch源码，还是不太方便的，换一个环境，换一个机器，都得操作一遍，希望官方早日解决这个问题。

### 3. 相关Issues
+ <https://github.com/pytorch/pytorch/issues/122321>
+ https://github.com/pytorch/pytorch/issues/122865
+ https://github.com/pytorch/pytorch/issues/99701
+ https://github.com/pytorch/pytorch/issues/117209

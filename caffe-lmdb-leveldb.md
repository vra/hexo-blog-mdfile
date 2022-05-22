title: Caffe中lmdb和leveldb格式数据的读取
date: 2017-10-01 17:50:28
tags:
 - Caffe
 - lmdb
 - leveldb
---
### 概述
Caffe里面的一种数据存储和读取方式是使用数据库格式，将数据保存到特定的一个数据库文件中，然后在代码里面整个读入这个数据库文件。Caffe支持的数据库格式包括lmdb和leveldb，可能很多人是因为caffe才知道这两个库的，但其实这两个库也是非常出名的工具。下面就展示下在Caffe里面用Python接口调用生成的LMDB或者LEVELDB格式的文件的代码吧。
<!--more-->
### LMDB 操作方式
具体方式见如下代码:
```python
import lmdb
env = lmdb.open('pool5-lmdb', readonly=True)
txn = env.begin()
for k, v in txn.cursor():
	print k,v

cur = txn.cursor()
k, v = cur.item()
print k,v
v = txn.get(k)
print v

import sys
sys.path.insert(0, '/data2/yunfeng/caffe20161019/python/')
import caffe
datum = caffe.proto.caffe_pb2.Datum()
datum.ParseFromString(v)
print datum.label, datum.channels, datum.width, datum.height

import numpy as np
data = caffe.io.datum_to_array(datum)
print data.shape
```

### LEVELDB 操作方式：

```python
import leveldb
db = leveldb.LevelDB('pool5-leveldb')
for k, v in db.RangeIter():
	print k,v

v = db.Get(k)
db.Put('new_key', 'new_value')
db.Delete('new_key')

batch = leveldb.WriteBatch();
batch.Put('hello', 'world');
batch.Put('hello again', 'world');
batch.Delete('hello');
db.Write(batch, sync = True);

import sys
sys.path.insert(0, '/data2/yunfeng/caffe20161019/python/')
import caffe
datum = caffe.proto.caffe_pb2.Datum()
datum.ParseFromString(v)
print datum.label, datum.channels, datum.width, datum.height

import numpy as np
data = caffe.io.datum_to_array(datum)
print data.shape
```

---
title: Python Multiprocessing使用Queue的例子
date: 2022-05-16 22:10:55
tags:
- Python
- 多进程
- Linux
---
对于一些计算密集性的任务，使用Python的多进程能显著缩短运行的时间。例如对10个元素进行相同的操作，通过Python的`multiprocessing` 包可以进行并行化，实测能有数倍的速度提升。这里写一个简单的例子，将所有的结果写入队列，等队列拿到10个结果后，将结果写入文件。
<!--more-->

```python
from multiprocessing import Queue, Process, Pool
import os
import time

import numpy as np


def write_queue(q, i):
    print(f'Begin process ({os.getpid()})')
    cur_value = i * i

    q.put(cur_value)


def read_queue(q, num_sample):
    val_list = []
    while True:
        v = q.get(True)
        val_list.append(v)

        if len(val_list) >= num_sample:
            np.save('data.npy', np.array(val_list))
            return


def main():
    NUM_PROCESS = 10
    q = Queue(NUM_PROCESS)
    pws = []
    for i in range(NUM_PROCESS):
        pw = Process(target=write_queue, args=(q, i))
        pws.append(pw)

    pr = Process(target=read_queue, args=(q, NUM_PROCESS))

    for pw in pws:
        pw.start()

    pr.start()
    for pw in pws:
        pw.join()

    pr.terminate()

    data = np.load('data.npy')
    print(data)


if __name__ == '__main__':
    main()
```

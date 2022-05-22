---
title: 检测Python代码中没有用到的函数和变量
date: 2021-11-07 19:57:22
tags:
- Python
---
在重构Python代码的时候，需要统计有哪些函数和变量没有用到，搜索后发现一个简单的工具[vulture](https://pypi.org/project/vulture/)，可以完成这个功能。 

操作也很简单, pip 安装包：
```bash
pip install vulture
```

检测代码：
```bash
vulture tester.py
```

输出大概是这样:
```bash
tester.py:19: unused import 'time' (90% confidence)
tester.py:181: unused variable 'raw_img' (100% confidence)
tester.py:300: unused method 'run_on_video' (60% confidence)
tester.py:403: unused method 'render_results' (60% confidence)
```
可以看到，每一行是一个检测结果，包含文件名称，行数，检测结果以及检测的置信度，可以根据这个输出来重构代码。

## 参考：
1. <https://stackoverflow.com/questions/693070/how-can-you-find-unused-functions-in-python-code>

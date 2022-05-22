---
title: Python 彩色命令行输出
date: 2019-09-10 16:06:21
tags:
 - Python
 - coloredlogs
 - Linux
---
效果：
![](/imgs/coloredlogs-demo.png)
下面描述如何来实现。
<!--more-->

安装coloredlogs:
```bash
pip3 install --user coloredlogs
```

就可以了，使用示例：
```python
import logging

import coloredlogs


FIELD_STYLES = dict(
    asctime=dict(color='green'),
    hostname=dict(color='magenta'),
    levelname=dict(color='green', bold=coloredlogs.CAN_USE_BOLD_FONT),
    filename=dict(color='magenta'),
    name=dict(color='blue'),
    threadName=dict(color='green')
)

LEVEL_STYLES = dict(
    debug=dict(color='green'),
    info=dict(color='cyan'),
    warning=dict(color='yellow'),
    error=dict(color='red'),
    critical=dict(color='red', bold=coloredlogs.CAN_USE_BOLD_FONT)
)

logger = logging.getLogger('tos')
coloredlogs.install(
    level="DEBUG",
    fmt="[%(levelname)s] [%(asctime)s] [%(filename)s:%(lineno)d] %(message)s",
    level_styles=LEVEL_STYLES,
    field_styles=FIELD_STYLES)

logger.debug('This is Debug mode')
logger.info('This is info mode')
logger.warn('This is warn mode')
logger.error('This is error mode')
logger.critical('This is critical mode')
```

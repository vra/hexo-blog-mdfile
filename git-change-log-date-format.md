---
title: git log 修改提交日期显示格式
date: 2022-03-26 21:34:49
tags:
- Linux
- Git
---
git log 默认显示的日期格式是欧美形式的，使用起来不太习惯，还得在大脑中进行一次转换，有点费脑。后来发现有办法可以修改日期显示格式，在git仓库下执行下面的命令即可：
```bash
git config log.date iso
```
如果要对所有的git仓库都起作用，添加`--global`选项即可。

另外输入`git config` 按tab键，可以显示所有的配置选项。

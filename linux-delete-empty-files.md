---
title: Linux小技巧：使用find命令来删除空文件
date: 2023-01-20 14:36:13
tags:
- Linux
- find
- 总结
---
在某个目录下有很多代码创建的空文件，分布在不同层级的子目录中，我们有没有办法可以快速地全部把它们删掉呢？

[find](https://man7.org/linux/man-pages/man1/find.1.html)是Linux系统中的一个强大的命令，通过它我们可以找到空文件，然后将它们进行删除。

TL;DR
最终命令如下：
```bash
find . -type f -size 0 -print -delete
```

几个参数详细的说明见下。
<!--more-->

`-type`表示匹配项的文件类型，`d`表示文件夹，`f`表示文件，`l`表示软链接等，完整的类型如下:
```plain
b: block (buffered) special

c: character (unbuffered) special

d: directory

p: named pipe (FIFO)

f: regular file

l: symbolic link; this is never true if the -L option
 : or the -follow option is in effect, unless the
 : symbolic link is broken.  If you want to search for
 : symbolic links when -L is in effect, use -xtype.

s: socket
```

所以下面的命令只会列出当前目录下的所有文件:
```bash
find . -type f
```
`-size`用来进行文件和目录的大小判断，例如`-size 6c`表示大小等于6字节，`-size -6c`表示小于6字节，`-size +6c`表示大于6字节，大小单位包括：c：字节，w:双字节，k:1024字节，M：1024*1024字节，G：1024*1024*1024字节，不加单位的话，等于b:512字节:
```bash
# 寻找当前目录下大小为0的文件或目录
find . -size 0

# 寻找当前目录下小于512字节的文件或目录
find . -size -1

# 寻找当前目录下大于1字节的文件或目录
find . -size +1c

# 寻找当前目录下大于1M的文件或目录
find . -size +1M
```
有了这个选项，就能很容易地过滤出当前目录下的空文件了:
```bash
find . -type f -size 0
```

另一个选项是`-delete`，它的作用是直接删除找到的文件。

还有一个选项是`-print`，即打印匹配的文件路径到标准输出。

结合这几个选项，我们就能删除当前目录下的所有空文件，并且在删除时打印文件名：
```bash
find . -type f -size 0 -print -delete
```

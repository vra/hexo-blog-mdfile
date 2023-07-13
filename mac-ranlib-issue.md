---
title: mac 编译问题解决——building for macOS-x86_64 but attempting to link with file built for xxx
date: 2023-05-27 01:17:15
tags:
- 总结
- macOS
- GNU
- Apple 
- XCode
- ranlib
- binutils
- TVM
---

在编译TVM的一个[fork版本](https://github.com/mlc-ai/relax)时，遇到下面的报错：
>ld: warning: ignoring file libbacktrace/lib/libbacktrace.a, building for macOS-x86_64 but attempting to link with file built for unknown-unsupported file format ( 0x21 0x3C 0x61 0x72 0x63 0x68 0x3E 0x0A
 0x2F 0x20 0x20 0x20 0x20 0x20 0x20 0x20 )
Undefined symbols for architecture x86_64:
  "_backtrace_create_state", referenced from:
      __GLOBAL__sub_I_logging.cc in logging.cc.o
  "_backtrace_full", referenced from:
      tvm::runtime::Backtrace() in logging.cc.o
  "_backtrace_syminfo", referenced from:
      tvm::runtime::(anonymous namespace)::BacktraceFullCallback(void*, unsigned long, char const*, int, char const*) in logging.cc.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make[3]: *** [libtvm_runtime.dylib] Error 1
make[2]: *** [CMakeFiles/tvm_runtime.dir/all] Error 2
make[2]: *** Waiting for unfinished jobs....

搜索了一下，发现核心原因是Mac下ranlib命令采用了GNU版本，而非Apple版本导致的，下面详细展开报错原因和解决办法。
<!--more-->
在Mac下，有两套编译工具链，GNU的和Apple（通过Xcode安装）的，GNU的以`gcc`为代表，而Apple的则以`clang`为代表，在这两个核心编译工具周围，又有很多别的小的编译工具。

通过log输出发现，编译工具用的是`/usr/bin/cc`, 执行`/usr/bin/cc --version` 命令，输出如下：
```bash
$ /usr/bin/cc --version
Apple clang version 14.0.0 (clang-1400.0.29.202)
Target: x86_64-apple-darwin22.2.0
Thread model: posix
InstalledDir: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin
```
可以看到是Apple的编译工具链Apple clang。

在编译过程中，发现log中有下面的输出：
```plain
ibtool: install: ranlib /path/to/relax/build/libbacktrace/lib/libbacktrace.a
```
可以看到调用了`ranlib`命令来生成`libbacktrace.a`。

通过`which ranlib` 验证ranlib的路径：
```bash
$ which ranlib
/usr/local/opt/binutils/bin/ranlib

$ ranlib --version
GNU ranlib (GNU Binutils) 2.40
Copyright (C) 2023 Free Software Foundation, Inc.
This program is free software; you may redistribute it under the terms of
the GNU General Public License version 3 or (at your option) any later version.
This program has absolutely no warranty.
```
可以看到，找到的是GPN版本的ranlib，而不是跟编译工具匹配的Apple的ranlib（路径是`/usr/bin/ranlib`)。

如果是Apple的ranlib工具的话，`ranlib --version`输出应该是下面这样：
```bash
$ranlib --version
error: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/ranlib: unknown option character `-' in: --version
Usage: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/ranlib [-sactfqLT] [-] archive [...]
```


那为什么会有两套工具链混合使用导致出错的问题？这是因为路径设置优先级的原因，在PATH中，`/usr/local/opt/binutils/bin`在`/usr/bin`的前面：

```bash
$ echo $PATH
...:/usr/local/opt/binutils/bin:/usr/bin:...
```

所以在搜索可执行文件时，先找到了GNU的ranlib，而这个又与Apple的编译工具链不兼容。导致编译出错。

那`ranlib`是干什么用的呢？根据ChatGPT， ranlib功能如下：
> ranlib是一个命令行工具，用于在静态库中创建索引（也称为符号表）。索引提供静态库中所有符号（函数、变量等）的列表。它帮助编译器和链接器在链接时更快地查找和解析符号。当一个程序需要链接静态库时，链接器会使用ranlib创建的索引来确定静态库中包含的符号，以便正确地链接程序。

可以看到，ranlib对于编译静态库来说，是必不可少的（与`ar -s`完全等效）。

其实我不记得在PATH中添加过`/usr/local/opt/binutils/bin`这个目录，应该是安装某些包后自动更新的。

那这个问题该怎么解决呢？通过上面的分析，我们也能发现其实解决办法也比较直观，总体来说有两种，一种是修改PATH中两个目录的寻找优先级，保证先找到的是Apple的工具，也就是`/usr/bin`目录在`/usr/local/opt` 前面；另一种是直接卸载GNU的工具`binutils`，这样就不会有冲突。

在这里我选择执行第二种，具体命令为：
```bash
$ brew uninstall binutils
```
然后再检查`ranlib --version` 命令的输出，确认是Apple的工具链后再`make clean`，重新编译即可。

### 参考
1. https://stackoverflow.com/a/72904009
2. https://github.com/bitcoin/bitcoin/issues/20825#issuecomment-753444519


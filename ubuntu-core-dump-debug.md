title: 在ubuntu中进行core dump调试
date: 2017-12-03 23:48:55
tags:
 - Linux
 - gdb
 - gcc
 - g++
 - Ubuntu
---
在Linux环境下执行程序的时候，有的时候会出现段错误（‘segment fault’），同时显示core dumped,就像下面这样：
```bash
[1]    15428 segmentation fault (core dumped)  ./a.out
```
下面是我网上找到的段错误的定义和说明：  

	A segmentation fault (often shortened to segfault) is a particular error condition that can occur during the operation of computer software. In short, a segmentation fault occurs when a program attempts to access a memory location that it is not allowed to access, or attempts to access a memory location in a way that is not allowed (e.g., attempts to write to a read-only location, or to overwrite part of the operating system). Systems based on processors like the Motorola 68000 tend to refer to these events as Address or Bus errors.

	Segmentation is one approach to memory management and protection in the operating system. It has been superseded by paging for most purposes, but much of the terminology of segmentation is still used, "segmentation fault" being an example. Some operating systems still have segmentation at some logical level although paging is used as the main memory management policy.

	On Unix-like operating systems, a process that accesses invalid memory receives the SIGSEGV signal. On Microsoft Windows, a process that accesses invalid memory receives the STATUS_ACCESS_VIOLATION exception.

简单理解就是访问了不该访问的内存就会产生段错误。  
而core dump是一种将出错时的调用堆栈等信息写入到一个文件中，方便后面调试。Ubuntu下需要进行一些设置才能正确地调试core dump，下面是详细的说明。  
<!--more-->
### ulimit 设置
ulimit是对shell启动进程所占系统资源进行限制的一个工具，详细的使用说明可以看[这里](https://www.ibm.com/developerworks/cn/linux/l-cn-ulimit/)。在这里我们需要对ulimit进行设置，因为在Ubuntu下，默认的core 文件的大小是0，可以通过执行`ulimit -a`查看所有的选项设置值：
```bash
$ ulimit -a
-t: cpu time (seconds)              unlimited
-f: file size (blocks)              unlimited
-d: data seg size (kbytes)          unlimited
-s: stack size (kbytes)             8192
-c: core file size (blocks)         0
-m: resident set size (kbytes)      unlimited
-u: processes                       2063144
-n: file descriptors                1024
-l: locked-in-memory size (kbytes)  64
-v: address space (kbytes)          unlimited
-x: file locks                      unlimited
-i: pending signals                 2063144
-q: bytes in POSIX msg queues       819200
-e: max nice                        0
-r: max rt priority                 0
-N 15:                              unlimited
```
可以看到上面的结果中`-c: core file size (blocks)` 那项的值是0，因此在段错误发生core dump的时候，默认也不会生成core文件。  
那应该怎么修改core文件的大小呢？`ulimit`的值都可以通过`ulimit -k v`的形式来设置，其中`-k`就是上面结果中的第一列，而`v`就是设置的值，最大可以设置为`unlimited`，所以我们可以这样来设置:
```bash
ulimit -c unlimited
```
再用 `ulimit -a`查看，发现对`-c`的修改已经生效了：
```bash
$ ulimit -a
-t: cpu time (seconds)              unlimited
-f: file size (blocks)              unlimited
-d: data seg size (kbytes)          unlimited
-s: stack size (kbytes)             8192
-c: core file size (blocks)         unlimited
-m: resident set size (kbytes)      unlimited
-u: processes                       2063144
-n: file descriptors                1024
-l: locked-in-memory size (kbytes)  64
-v: address space (kbytes)          unlimited
-x: file locks                      unlimited
-i: pending signals                 2063144
-q: bytes in POSIX msg queues       819200
-e: max nice                        0
-r: max rt priority                 0
-N 15:                              unlimited
```
	注意：新开一个Shell的时候，ulimit选项都恢复了默认选项，需要重新设置该值


### 设置`core_pattern`
出了上面的ulimit设置，我们还需要设置`core_pattern`，即发送core dump后，对core文件执行什么操作，这个可以通过查看`/proc/sys/kernel/core_pattern`文件来得到，在Ubuntu 16.04上面，上述文件内容如下：
```bash
$ cat /proc/sys/kernel/core_pattern
|/usr/share/apport/apport %p %s %c %P
```
其中的`l`表示执行后面的命令，而后面的`apport`是Ubuntu的bug反馈的工具，因此在Ubuntu下，默认的core dump 段错误处理机制是将其作为一个bug，进行bug检查，如果是bug的话就进行上报。
在这种设定下，我们没法用gdb来调试我们程序的错误。  
因此这里我们得修改`core_pattern`的内容，将其修改为`core`即可。但是没有找到修改`core_pattern`文件的方式，因为它本身不是一个实体的文件，所以这里有个小技巧来实现这个功能：暂停`apport`服务：
```bash
sudo service apport stop
```
然后查看`core_pattern`的内容：
```bash
$ cat /proc/sys/kernel/core_pattern
core
```
可以看到这里的操作以及改成`core`，即生成core文件。  

### gcc/g++设置debug模式
除了上面的两项设置，这里还需要在编译代码的时候通过加`-g`参数来启用debug模式，这样会在生成的可执行文件中加入调试信息：
```bash
g++ -g xxx.cpp
gcc -g xxx.c
```
### 采用gdb来调试程序
完成上面的设置之后，就可以使用gdb来调试了，当程序发生段错误，而且core文件也生成后，通过执行下面的命令来开始调试：
```
gdb ./a.out core
```
其中`./a.out`是到可执行文件的路径，而`core`是core dump生成的文件。  
之后执行在gdb调试环境里面执行`bt`命令，即可定位到报错的位置，然后再根据报错信息，利用搜索引擎查找解决方法。下面是我的一个调试现场信息：
```bash
$ gdb ./build/tools/caffe.bin core
GNU gdb (Ubuntu 7.11.1-0ubuntu1~16.5) 7.11.1
Copyright (C) 2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./build/tools/caffe.bin...(no debugging symbols found)...done.
[New LWP 38035]
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Core was generated by `./build/tools/caffe.bin'.
Program terminated with signal SIGSEGV, Segmentation fault.
#0  0x00007f99b6ff5f66 in google::protobuf::Arena::AllocateAligned(std::type_info const*, unsigned long) () from /usr/local/lib/libopencv_dnn.so.3.3
(gdb) bt
#0  0x00007f99b6ff5f66 in google::protobuf::Arena::AllocateAligned(std::type_info const*, unsigned long) () from /usr/local/lib/libopencv_dnn.so.3.3
#1  0x00007f99b6ff5f89 in google::protobuf::Arena::AddListNode(void*, void (*)(void*)) () from /usr/local/lib/libopencv_dnn.so.3.3
#2  0x00007f99b70473ae in google::protobuf::FileDescriptorProto::New(google::protobuf::Arena*) const [clone .localalias.409] () from /usr/local/lib/libopencv_dnn.so.3.3
#3  0x00007f99b6147038 in google::protobuf::MessageLite::ParseFromArray(void const*, int) () from /usr/lib/x86_64-linux-gnu/libprotobuf.so.9
#4  0x00007f99b61901b6 in google::protobuf::EncodedDescriptorDatabase::Add(void const*, int) () from /usr/lib/x86_64-linux-gnu/libprotobuf.so.9
#5  0x00007f99b61519b8 in google::protobuf::DescriptorPool::InternalAddGeneratedFile(void const*, int) () from /usr/lib/x86_64-linux-gnu/libprotobuf.so.9
#6  0x00007f99b618048c in google::protobuf::protobuf_AddDesc_google_2fprotobuf_2fdescriptor_2eproto() () from /usr/lib/x86_64-linux-gnu/libprotobuf.so.9
#7  0x00007f99b7e346ba in call_init (l=<optimized out>, argc=argc@entry=1, argv=argv@entry=0x7ffe8743a4d8, env=env@entry=0x7ffe8743a4e8) at dl-init.c:72
#8  0x00007f99b7e347cb in call_init (env=0x7ffe8743a4e8, argv=0x7ffe8743a4d8, argc=1, l=<optimized out>) at dl-init.c:30
#9  _dl_init (main_map=0x7f99b804b168, argc=1, argv=0x7ffe8743a4d8, env=0x7ffe8743a4e8) at dl-init.c:120
#10 0x00007f99b7e24c6a in _dl_start_user () from /lib64/ld-linux-x86-64.so.2
#11 0x0000000000000001 in ?? ()
#12 0x00007ffe8743c1cb in ?? ()
#13 0x0000000000000000 in ?? ()
(gdb) 
```

### 参考
1. <https://www.ibm.com/developerworks/cn/linux/l-gccrtl/index.html>
2. <https://www.ibm.com/developerworks/cn/linux/l-cn-ulimit/>










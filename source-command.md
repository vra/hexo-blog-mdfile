title: Linux 下的source命令学习
date: 2015-07-03 19:43:56
tags:
 - Linux
 - Shell 
---
前些天在装opencl的[beignet](http://www.freedesktop.org/wiki/Software/Beignet/)实现版本时，发现wiki中里面有个点命令.，不知道具体含义就百度了下，结果学了一些相关的知识，记录如下。

<!--more-->

##1.概述

`source`命令是bash的内置命令，与点命令.等效，唯一不同的是点命令是[在POXIS下定义的]](http://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#dot)。`source`命令的执行格式是`source script`，是在当前shell进程中依次执行script文件中的语句。那么与普通的 `sh script`和`./script`有什么不同呢？主要有两个不同点：

 1. `source` 的执行是在当前进程中执行，而`sh script`和`./script`在执行的时候，当前进程会开辟一个新的子进程，然后在子进程中执行script中的语句。
 2. 使用`source`命令的文件不需要有执行权限，而./script方式执行的方式需要script文件有可执行权限（注意：`sh script` 不需要script文件有可执行权限）。

##2. 测试实例
我们可以举几个例子来展示上面提到的不同点(例子都摘自参考内容中的第1个链接)。

###实例1

编写脚本test.sh如下：

```bash
echo $$
```

需要说明一下，在Linux中，每个进程都有一个独一无二的进程号，简称为`PID`。而`$$`就表示当前进程的`PID`。所以上述脚本的作用就是输出当前进程的`PID`。
我们可以用两种方式来执行这个脚本，先使用`source`命令来执行：

```bash
> source test.sh
3824
> source test.sh
3824
> source test.sh
3824
```

可以看到每次输出的结果都是3824。然后使用`sh script`方式来执行：

```bash
> sh test.sh
3884
> sh test.sh
3889
> sh test.sh
3894
```

可以看到每次输出的结果都在改变。
这个测试说明：使用`source`命令在当前进程执行，而使用`sh script`命令则每次执行时都生成不同的子进程，在子进程中执行，执行完后面文件中的指令后再返回主进程。

###实例2

编写测试脚本`test.sh`:

```bash
echo "FOO:"$(env | grep FOO)
export FOO=foo
echo "FOO:"$(env | grep FOO)
echo "PWD:"$PWD
cd mydir
echo "PWD:"$PWD
```

这个脚本先是测试环境变量中是否包含名为`FOO`的环境变量，然后新建的环境变量`Foo=foo`。然后是输出当前所在目录，接着切换到当前目录的`mydir`子目录，然后再输出当前所在目录。
在执行这个脚本前，我们先检查下当前环境:

```bash
> env | grep FOO
> echo $PWD
/home/yunfeng
```

这说明当前环境中没有名为`FOO`的变量，当前所在路径为`/home/yunfeng`。
然后我们执行`sh test.sh`，输出为：

```bash
FOO:
FOO:FOO=foo
PWD:/home/yunfeng
PWD:/home/yunfeng/mydir
```

然后我们再检查下当前环境：

```bash
> env | grep FOO
> echo $PWD
/home/yunfeng
```

这说明使用`sh test.sh`执行的时候，并没有改变当前进程的环境变量和所在路径，而只是改变了新建的子进程的环境变量和所在路径。此外我们还可以得出结论：当前进程新建shell子进程的时候为子进程复制了当前进程的环境变量（包括路径）。
然后使用`source`命令执行`test.sh`:

```bash
source test.sh
FOO:
FOO:FOO=foo
PWD:/home/yunfeng
PWD:/home/yunfeng/mydir
```

然后检查当前环境：(其实可以发现，当前目录已经切换到`mydir`了)


```bash
> env | grep FOO
FOO=foo
> echo $PWD
/home/yunfeng/mydir
```

可以看出使用`source`命令时，会改变当前进程的环境变量。


title: argparse简要用法总结
date: 2017-12-02 18:51:05
tags:
 - Python
 - Linux
---
[argparse](https://docs.python.org/3/library/argparse.html) 是python自带的命令行参数解析包，可以用来方便地读取命令行参数，当你的代码需要频繁地修改参数的时候，使用这个工具可以将参数和代码分离开来，让你的代码更简洁，适用范围更广。  
argparse使用比较简单，常用的功能可能较快地实现出来，下面我分几个步骤，**以Python3为例**，逐渐递增地讲述argparse的用法。  
<!--more-->

### 1. 基本框架
下面是使用argparse从命令行获取用户名，然后打印'Hello '+ 用户名，假设python文件名为`print_name.py`:
```python
# file-name:print_name.py
import argparse

def get_parser():
    parser = argparse.ArgumentParser(description="Demo of argparse")
    parser.add_argument('--name', default='Great')
    
    return parser


if __name__ == '__main__':
    parser = get_parser()
    args = parser.parse_args()
    name = args.name
    print('Hello {}'.format(name))
```
在命令行执行如下命令：
```bash
$ python print_name.py --name Wang
Hello Wang
```
上面的代码段中，我们显示引入了`argparse`包，然后通过`argparse.ArgumentParser`函数生成argparse对象，其中这个函数的`description`函数表示在命令行显示帮助信息的时候，这个程序的描述信息。之后我们通过对象的`add_argument`函数来增加参数。这里我们只增加了一个`--name`的参数，然后后面的`default`参数表示如果没提供参数，我们默认采用的值。即如果像下面这样执行命令：
```bash
$ python print_name.py 
```
则输出是:
```bash
$ Hello Great
```
最后我们通过argpaser对象的`parser_args`函数来获取所有参数`args`，然后通过`args.name`的方式得到我们设置的`--name`参数的值，可以看到这里argparse默认的参数名就是`--name`形式里面`--`后面的字符串。  
整个流程就是这样，下面我们详细讲解`add_argument`函数的一些最常用的参数，使得你看完这个教程之后，能完成科研和工作中的大部分命令解析任务。  

### 2. `default`：没有设置值情况下的默认参数
如同上例中展示的，default表示命令行没有设置该参数的时候，程序中用什么值来代替。

### 3. `required`: 表示这个参数是否一定需要设置
如果设置了`required=True`,则在实际运行的时候不设置该参数将报错：
```python
...
parser.add_argument('-name', required=True)
...
```
则运行下面的命令会报错：
```bash
$ python print_name.py
usage: print_name.py [-h] --name NAME
print_name.py: error: argument --name is required
```

### 4. `type`：参数类型
默认的参数类型是str类型，如果你的程序需要一个整数或者布尔型参数，你需要设置`type=int`或`type=bool`，下面是一个打印平方的例子：
```python
#name: square.py
import argparse

def get_parser():
    parser = argparse.ArgumentParser(
        description='Calculate square of a given number')
    parser.add_argument('-number', type=int)

    return parser


if __name__ == '__main__':
    parser = get_parser()
    args = parser.parse_args()
    res = args.number ** 2
    print('square of {} is {}'.format(args.number, res))
```
执行：
```bash
$ python square.py -number 5
square of 5 is 25 
```

### 5. `choices`：参数值只能从几个选项里面选择
如下面的代码：
```python
# file-name: choices.py
import argparse

def get_parser():
    parser = argparse.ArgumentParser(
        description='choices demo')
    parser.add_argument('-arch', required=True, choices=['alexnet', 'vgg'])

    return parser

if __name__ == '__main__':
    parser = get_parser()
    args = parser.parse_args()
    print('the arch of CNN is '.format(args.arch))
```
如果像下面这样执行会报错：
```bash
$ python choices.py -arch resnet
usage: choices.py [-h] -arch {alexnet,vgg}
choices.py: error: argument -arch: invalid choice: 'resnet' (choose from 'alexnet', 'vgg')
```
因为我们所给的`-arch`参数`resnet`不在备选的`choices`之中，所以会报错

### 6. `help`：指定参数的说明信息
在现实帮助信息的时候，help参数的值可以给使用工具的人提供该参数是用来设置什么的说明，对于大型的项目，help参数和很有必要的，不然使用者不太明白每个参数的含义，增大了使用难度。  
下面是个例子：
```python
# file-name: help.py
import argparse

def get_parser():
    parser = argparse.ArgumentParser(
        description='help demo')
    parser.add_argument('-arch', required=True, choices=['alexnet', 'vgg'],
        help='the architecture of CNN, at this time we only support alexnet and vgg.')

    return parser


if __name__ == '__main__':
    parser = get_parser()
    args = parser.parse_args()
    print('the arch of CNN is '.format(args.arch))
```
在命令行加`-h`或`--help`参数运行该命令，获取帮助信息的时候，结果如下：
```bash
$ python help.py -h
usage: help.py [-h] -arch {alexnet,vgg}

choices demo

optional arguments:
  -h, --help           show this help message and exit
  -arch {alexnet,vgg}  the architecture of CNN, at this time we only support
                       alexnet and vgg.
```
### 7. `dest`：设置参数在代码中的变量名
argparse默认的变量名是`--`或`-`后面的字符串，但是你也可以通过`dest=xxx`来设置参数的变量名，然后在代码中用`args.xxx`来获取参数的值。

### 8. `nargs`： 设置参数在使用可以提供的个数
使用方式如下：
```python
parser.add_argument('-name', nargs=x)
```
其中`x`的候选值和含义如下：
```bash
值  含义
N   参数的绝对个数（例如：3）
'?'   0或1个参数
'*'   0或所有参数
'+'   所有，并且至少一个参数
```
如下例子：
```python
# file-name: nargs.py
import argparse

def get_parser():
    parser = argparse.ArgumentParser(
        description='nargs demo')
    parser.add_argument('-name', required=True, nargs='+')

    return parser


if __name__ == '__main__':
    parser = get_parser()
    args = parser.parse_args()
    names = ', '.join(args.name)
    print('Hello to {}'.format(names))
```
执行命令和结果如下：
```bash
$ python nargs.py -name A B C
Hello to A, B, C
```

参考链接：
 1. <http://blog.xiayf.cn/2013/03/30/argparse/>
 2. <https://docs.python.org/3/library/argparse.html>

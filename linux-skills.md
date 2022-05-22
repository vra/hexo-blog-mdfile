title: Linux 小技巧总结
date: 2017-10-29 17:20:39
tags:
 - Linux
 - 总结
---
## 概述
这里列举了我常用的一些Linux命令行下的技巧，希望对大家有帮助。
<!--more-->

### 1. 按行合并2个文件
即第一个文件的第一行接第二个文件的第一行，然后是第一个文件的第二行和第二个文件的第二行，举例：
a.txt
```bash
1
2
3
```
b.txt
```bash
a
b
c
```
期望的结果：
```bash
1
a
2
b
3
c
```
命令：
```bash
paste -d '\n' a.txt b.txt > c.txt
```


### 2. 删除行尾多余的`\r`
一般在Windows平台创建的文件行尾有多余的`\r`，在Linux命令行操作的时候会报错。
命令：
```bash
# method 1:
tr -d '\r' < infile > outfile

# method 2:
cat infile |sed 's/\r$//g' >> outfile
```

### 3. 批量创建目录
例如我想创建dir1，dir2, ... dir100的目录。  
命令：
```bash
mkdir dir{1..100} # 创建目录dir1, dir2, dir3, ... dir100
mkdir {1..100} # 创建目录1，2，3，... 100
```

### 4. 删除特定目录的所有空文件或空目录
命令：
```bash
# 删除空目录
find /path/to/target/dir -type d -empty -exec rmdir {} \;
# 删除空文件
find /path/to/target/dir -type f -empty -exec rm {} \;
```

### 5. 删除名字中包含特殊字符的文件或目录
在rm后面加`--`，然后对文件名加双引号。  
命令：
```bash
rm -- "--abc"
rm -- ")abc"
```

### 6. 查找目录下所有文件中包含特定字符串的文件
如要搜索当前目录中包含`include`的所有文件。  
命令：
```bash
grep -RIn "include" * # R：递归搜索，I:区分大小写，n:显示行号
```

### 7. ls命令只列出目录/文件
命令:
```bash
# 只列出目录
ls  -l /path/to/dir |grep '^d' 
# 只列出文件
ls  -l /path/to/dir |grep '^-' 
```

### 8. 统计目录下的文件和目录数
命令：
```bash
ls /pat/to/dir |wc -l
```
### 9. 按数字顺序对文件每行排序
如文件a.txt如下：
```bash
10
12
13
1
2
15
16
```
期望得到的结果文件b.txt:
```bash
1
2
10
12
13
15
16
```
命令：
```bash
sort -n a.txt > b.txt
```

### 10. 显示当前系统安装的lib文件的版本
命令:  
```bash
ldconfig -v |grep libname
```

### 11. 显示系统所有安装的软件包
```bash
apt list --installed # 不加--installed 选项则会列出源里面的所有包名
```

### 12. 查看硬盘读写情况
```bash
iostat 
```

### 13. 动态地查看某个命令的输出结果
```bash
watch -n 1 nvidia-smi # -n 后面的参数表示刷新的秒数
```

### 14. 删除某个文件中所有不包含某个字段的行
如文件a.txt如下：  
```bash
ustc-123
ustc-222
ustc-bcd
fdu-222
pku-222
```
现在要删除不包含`ustc`的所有行，得到b.txt如下：
```bash
ustc-123
ustc-222
ustc-bcd
```
命令：  
```bash
sed '/ustc/!d' a.txt > b.txt
```

### 15. 删除某个文件中包含特定字段的所有行
```bash
sed '/ustc/d' a.txt > b.txt
```

### 16. 查找局域网里面所有的联网机器的IP
```bash
 sudo nmap -sP 192.168.110.0/24
```

### 17. 不同机器间同步数据
```bash
 rsync -aPhv src  --exclude "excluded_dir" user@host:~/dst
```
rsync命令的详细用法可以参考[这里](http://roclinux.cn/?p=2643)


### 18. 通过ssh在远端执行命令
例如我想在本地屏幕上显示在远程服务器上执行某条命令的结果。  
```bash
ssh user@host command
```

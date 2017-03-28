title: Vim 文本操作总结备忘
date: 2017-02-27 22:24:35
tags:
 - Vim
 - Linux
 - 总结
---
在学习和科研工作中，我使用Vim比较多，而且常常遇到处理文本的情况，比如删除文本中的空行，每行前面增加行号等等这些需求。我一般是直接取Google搜索，但是有的时候也不一定能快速地搜索到，所以这里我把常用到的需求和对应的Vim下的解决方法列出来，自己查起来方便些，也希望能帮助到别人。  
<!--more-->

下面我按每个需求来写，每条记录中，先是需求的介绍，然后是一个具体的例子，最后是解决方式。默认的解决方式是在Vim中的命令行模式下，按`：`后再敲入命令。  

### 1. 删除Vim中的空行
如下面的文本：
```bash
a

b
b

c

d
```
操作后空行被删去，变成下面这样：
```bash
a
b
b
c
d
```
解决方案：
```bash
:g/^$/d # 删除空白行，但是不删去包含withspace的行
:g/^\s*$/d # 删除空白行，包括删去包含withspace的行
```
参考链接：<http://stackoverflow.com/questions/706076/vim-delete-blank-lines>


### 2. 每行前面加行号
如原来文本如下：
```bash
a
b
b
c
```
则操作后变成：
```bash
1 a
2 b
3 b
4 c
```
解决方案：
```bash
:%s/^/\=printf('%d ', line('.'))
```
注意`%d`后面的空格，如果是要用点号`.`分割行号和内容的话，则将`%d `改成`%d.`即可。  

### 3. 重复每行中的某个部分
例如原来文本为：
```bash
name1/path1 
name2/path2
name3/path3
```
想要变成如下内容：
```bash
name1/path1 path1
name2/path2 path2
name3/path3 path3
```
即重复path部分。  
解决方案：  
这里的解决方法是找到需要重复的部分的特有的模式，通过正则表达式来匹配上，然后通过增加括号来引用。这个例子中，需要重复的部分的特征是前面有个`/`，所以可以通过匹配这个`/`来找到需要重复的部分。需要注意的是，`/`和`（`，`）`都需要进行转意，即在前面增加`\`。  
```bash
:0,$s/\/\(.\+\)/\/\1 \1/g
```


### 4. 在第i行最后插入数字i
原来文本：
```bash
user
user
user
user
```
期望的结果是：
```bash
user1
user2
user3
user4
```
解决方案：
```bash
:0,$s/$/\=prinf('%d', line('.'))
```

### 5. 对每行的数字进行特定的加减乘除操作
例如原先文本是这样：
```bash
wang 23
zhang 100
zhao 33
```
希望对每行的数字都加10,即最终的结果是：
```bash
wang 33
zhang 110
zhao 43
```
解决方案：
```bash
:%s/\d\+/\=submatch(0)+10
```
如果要进行减或者乘，则将上述命令中的最后面的加号改成减号和乘号即可，**对于除法，直接改乘除号不行，这里就只能通过乘以它的倒数来实现。**   
参考：<http://vim.1045645.n5.nabble.com/Subtract-integer-value-from-column-td1184983.html>


### 6. 生成与行号又特定关系的文本
例如要生成下面的文件：
```bash
1 test1_name1 100
2 test2_name2 200
3 test3_name3 300
4 test4_name4 400
```
解决方案：
```bash
:put=map(range(1,4), 'printf("%d test%d_name%d %d00",v:val,v:val,v:val,v:val)')
```
参考： <http://vim.wikia.com/wiki/Making_a_list_of_numbers>

### 7. 利用put函数生成等规律序列
例如想要生成如下序列：
```bash
PB11210245
PB11210246
PB11210247
PB11210248
PB11210249
PB11210250
PB11210251
PB11210252
PB11210253
PB11210254
PB11210255
```
解决方案：
```bash
:for i in range(245,255) | put='PB11210'.i |endfor
```

### 8. 只替换一行中的特定序号的匹配项
例如原来文本是这样：
```bash
a a a a a
```
替换奇数项为`b`，变成这样：
```bash
a b a b a 
```
解决方案：
```bash
:call feedkeys("nynyn") | s/a/b/gc
```
参考：<http://unix.stackexchange.com/questions/27178/vim-s-replace-first-n-g-occurrences-on-a-line>

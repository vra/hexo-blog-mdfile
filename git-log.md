---
title: git log 常见参数总结
date: 2022-03-26 20:59:55
tags:
- Linux
- Git
---
## 0. 概述
git log 是查看git提交记录的一个命令，它有非常多的控制参数和选项，合理使用的话，可以达到任何的精准控制目的。这里列一些日常使用可能会用到的用法，全部的用法，请在命令行`git help log`查看。
<!--more-->

## 1. 基本用法
### 1.1. 无参数
使用`git log`，会从新到旧显示所有的提交记录，按`j`往下翻页，按`k`往上翻页, 按`q`退出：
```bash
commit 869cc0a22aea80d34f0728e184842bdea42fe43b (HEAD -> master, origin/master, origin/HEAD)
Merge: 78aaac39 2e872840
Author: Johan Mabille <johan.mabille@gmail.com>
Date:   2021-04-10 02:29:06 +0200

    Merge pull request #2353 from JohanMabille/chunk

    Refactoring of xchunked_view

commit 2e872840a7ebc3e4e8b0f84cbae39360503243b1
Author: Johan Mabille <johan.mabille@gmail.com>
Date:   2021-04-09 16:59:16 +0200

    One xchunk_iterator to rule them all

commit 42fc49080522c94ea784541b53ef302ccb0344c0
Author: Johan Mabille <johan.mabille@gmail.com>
Date:   2021-04-08 22:40:14 +0200

    Refactoring of xchunked_view

	....
```
通过增加`-<n>`选项来显示最近n次的提交记录，如`git log -2`仅显示最近的2次提交；
```bash
commit 869cc0a22aea80d34f0728e184842bdea42fe43b (HEAD -> master, origin/master, origin/HEAD)
Merge: 78aaac39 2e872840
Author: Johan Mabille <johan.mabille@gmail.com>
Date:   2021-04-10 02:29:06 +0200

    Merge pull request #2353 from JohanMabille/chunk

    Refactoring of xchunked_view

commit 2e872840a7ebc3e4e8b0f84cbae39360503243b1
Author: Johan Mabille <johan.mabille@gmail.com>
Date:   2021-04-09 16:59:16 +0200

    One xchunk_iterator to rule them all
```

此外如果想显示每次提交代码修改的地方，可以增加`-p`参数:
```diff
commit 869cc0a22aea80d34f0728e184842bdea42fe43b (HEAD -> master, origin/master, origin/HEAD)
Merge: 78aaac39 2e872840
Author: Johan Mabille <johan.mabille@gmail.com>
Date:   2021-04-10 02:29:06 +0200

    Merge pull request #2353 from JohanMabille/chunk

    Refactoring of xchunked_view

commit 2e872840a7ebc3e4e8b0f84cbae39360503243b1
Author: Johan Mabille <johan.mabille@gmail.com>
Date:   2021-04-09 16:59:16 +0200

    One xchunk_iterator to rule them all

diff --git a/include/xtensor/xchunked_array.hpp b/include/xtensor/xchunked_array.hpp
index ed4003d0..23a843ec 100644
--- a/include/xtensor/xchunked_array.hpp
+++ b/include/xtensor/xchunked_array.hpp
@@ -126,10 +128,16 @@ namespace xt
         template <class S>
         const_stepper stepper_end(const S& shape, layout_type) const noexcept;

-        const shape_type& chunk_shape() const;
+        const shape_type& chunk_shape() const noexcept;
+        size_type grid_size() const noexcept;
+        const shape_type& grid_shape() const noexcept;
+
         chunk_storage_type& chunks();
         const chunk_storage_type& chunks() const;

+        chunk_iterator_type chunk_begin();
+        chunk_iterator_type chunk_end();
```

### 1.2. 显示统计信息
增加`--stat`选项可以显示某次提交文件的修改信息
```bash
commit 869cc0a22aea80d34f0728e184842bdea42fe43b (HEAD -> master, origin/master, origin/HEAD)
Merge: 78aaac39 2e872840
Author: Johan Mabille <johan.mabille@gmail.com>
Date:   2021-04-10 02:29:06 +0200

    Merge pull request #2353 from JohanMabille/chunk

    Refactoring of xchunked_view

commit 2e872840a7ebc3e4e8b0f84cbae39360503243b1
Author: Johan Mabille <johan.mabille@gmail.com>
Date:   2021-04-09 16:59:16 +0200

    One xchunk_iterator to rule them all

 include/xtensor/xchunked_array.hpp  |  45 ++++++++++-
 include/xtensor/xchunked_assign.hpp | 246 +++++++++++++++++++++++++++++++++++++++++++++-------------
 include/xtensor/xchunked_view.hpp   | 164 +++++++++++----------------------------
 3 files changed, 280 insertions(+), 175 deletions(-)
```

### 1.3. 过滤选项
默认所有的提交都显示，如果我们想搜索某段时间或某个人的提交记录，该怎么办呢？git提供了详细的命令来进行过滤，下面详细举例说明。
#### 1.3.1. 过滤作者
通过`--author`选项可以只显示某个人的提交记录，以这个仓库为例，下面的写法（FirstName，LastName, Email, FirstName + LastName, FirstName + LastName + Email）都可以:
```bash
git log --author=Johan
git log --author=Mabille
git log --author=johan.mabille@gmail.com
git log --author="Johan Mabille"
git log --author="Johan Mabille <johan.mabille@gmail.com>"
```
#### 1.3.2. 过滤代码关键字
通过`-S<keyword>`的形式可以搜索代码中增加或删除`keyword`的提交记录，比如`git log -Sxchunked_array`就会显示所有关于`xchunked_array`关键字的提交。结合前面的`-p`和`-<n>`参数，我们能很好的达到我们的搜索目的，比如只显示最近两次提交中关键词的修改内容：
```bash 
git log -Sxchunked_array -p -2
```
#### 1.3.3. 过滤提交信息中的关键字
此外还可以利用`--grep`选项来对commit内容进行过滤，比如我们想搜索所有包含`fix`的提交：
```bash
git log --grep fix
```
#### 1.3.4. 过滤日期
另一个很有用的选项是根据日期来过滤提交。日期过滤有好多形式，比如今年以来的提交，最近一周的提交，git提供了详细的控制命令，具体如下表:

|关键词|说明|例子|
|---|---|---|
|after=<xxx>|从xxx到现在的所有提交|after="2020-01-01"|
|since=<xxx>|从xxx到现在的所有提交，与after同义|since="2020-01-01"|
|before=<xxx>|xxx之前的所有提交|before="2020-01-01"|
|until=<xxx>|xxx之前的所有提交，与before同义|until="2020-01-01"|

日期格式如下：

|时间格式|说明|例子|
|---|---|---|
|YYYY-MM-DD|到某个具体日期的提交|since=2020-01-01| 
|n.minute|n分钟内的提交|since=3.minute| 
|n.hour|n小时内的提交|since=3.hour| 
|n.day|n天内的提交|since=3.day| 
|n.week|n周内的提交|since=3.week| 
|n.month|n个月内的提交|since=3.month| 
|n.year|n年内的提交|since=1.year| 
|组合|上述形式的组合|since=1.year,10.month| 

比如要显示2天内的所有提交，可以用下面的命令：
```bash
git log --since=2.day
```

## 2. 显示格式调整
默认的显示格式比较松散，一次提交占的空间太大，有没有办法显示地更紧凑呢？是有的，可以通过`--format=oneline`来设置：
```bash
869cc0a22aea80d34f0728e184842bdea42fe43b (HEAD -> master, origin/master, origin/HEAD) Merge pull request #2353 from JohanMabille/chunk
2e872840a7ebc3e4e8b0f84cbae39360503243b1 One xchunk_iterator to rule them all
42fc49080522c94ea784541b53ef302ccb0344c0 Refactoring of xchunked_view
....
```
这下每条记录在一行显示，包括提交hash串，commit信息。

那么 git 支持哪些format参数呢，总结下来如下表：

<table>
<tr><th>格式名称</th><th>格式说明</th></tr>

<tr><td><pre>oneline</pre></td><td>

```bash
<hash> <title-line>
```
</td></tr>

<tr><td><pre>short</pre></td><td>

```bash
commit <hash>
Author: <author>

<title-line>
```
</td></tr>


<tr><td><pre>medium</pre></td><td>

```bash
commit <hash>
Author: <author>
Date:   <author-date>

<title-line>

<full-commit-message>
```
</td></tr>


<tr><td><pre>full</pre></td><td>

```bash
commit <hash>
Author: <author>
Commit: <committer>

<title-line>

<full-commit-message>
```
</td></tr>


<tr><td><pre>fuller</pre></td><td>

```bash
commit <hash>
Author:     <author>
AuthorDate: <author-date>
Commit:     <committer>
CommitDate: <committer-date>

<title-line>

<full-commit-message>
```
</td></tr>

<tr><td><pre>reference</pre></td><td>

```bash
<abbrev-hash> (<title-line>, <short-author-date>)
```
</td></tr>

<tr><td><pre>email</pre></td><td>

```bash
From <hash> <date>
From: <author>
Date: <author-date>
Subject: [PATCH] <title-line>

<full-commit-message>
```
</td></tr>
</table>

还有一些别的选项，可以访问[这里](https://git-scm.com/docs/git-log#_pretty_formats)详细了解。

## 3. 自定义显示
上述命令在某些情况下可能并不能满足我们的需求，比如`--format=oneline`选项没有显示提交时间。因此我们需要自定义log显示的方式。git提供了对commit信息中各部分的描述符号，可以让我们方便地自定义log显示。

下面列出了常见的选项：

|选项|全称|含义|
|---|---|---|
|%cd|commit date|提交日期|
|%H|Hash|commit 的完整哈希串|
|%h|hash|commit 的简短哈希串|
|%an|author name|提交者名字|
|%ae|author email|提交者邮箱|
|%s|message|提交信息|

利用这些描述符，我们可以定制log显示格式，比如`git log --format="%cd|%h|%an|%ae|%s"` 就是显示提交日期，commit简短hash，提交者的名字和邮箱，以及提交内容：
```bash
2022-03-23 09:52:22 +0100|b2e23d05|Johan Mabille|johan.mabille@gmail.com|Merge pull request #2497 from spectre-ns/master
2022-03-18 20:59:53 -0300|a5a70449|spectre-ns|dahubley@hotmail.ca|Updated C++20 option for visual studio builds C++2a no longer a valid std option.
2022-03-18 10:59:57 +0100|f603205a|Johan Mabille|johan.mabille@gmail.com|Merge pull request #2496 from JohanMabille/adapt_doc
```

## 4. 命令组合
git log最强大的地方在于可以组合上述所有的选项，大大缩小搜索范围，能更方便地定位到想要的提交。例如我通过下面的命令，可以将搜索范围从3711条缩小到6条：
```bash
# 所有提交记录，共3177条
$ git log --oneline |wc
3177   19594  159959

# 添加搜索过滤，只剩6条
$ git log --since="2020-01-01" --until="2020-02-01" --grep fix --oneline
af5cc6c4 Merge pull request #1904 from BioDataAnalysis/emmenlau_tiny_variable_name_fix
0f3caa37 benchmark/CMakeLists.txt: fixed a tiny spelling mistake
218dcbe7 Merge pull request #1902 from kolibri91/fix_warning
38cb9617 Merge pull request #1886 from wolfv/fix_reshape_return
31cbd6d2 Merge pull request #1880 from wolfv/fix_older_cmake
f363e9d1 fix older cmak
```

title: 删除Git仓库中的大文件
date: 2018-05-20 08:51:22
tags:
 - Git
 - Linux
---
Git是用来管理源代码的一个工具，很多时候，我们不想让Git来跟踪较大的二进制文件。但是如果不小心将某个文件加入到Git的缓存区后，不管后面怎么删除这个大文件，Git始终都保存有这个文件的历史记录，因此项目会很大。拿下面例子来说，我们有个500M的文件`cnn.model`，通过下面的命令加入到git暂存区或提交到远端（提交时自动执行git gc命令，生成pack文件）：
```bash
$ git add cnn.model
$ git commit -m "add file cnn.model"
$ git push
```
经过这步操作，用`du -sh .`命令查看项目大小的话，发现足足有1000多M，因为本地文件`cnn.model`以及`.git`目录中的object也有一份这个文件的记录。  
即使使用`git rm`命令删除当前的`cnn.model`文件，`.git`目录中还是记录有这个大文件的记录，因此后面别人clone这个项目后，项目还是很大。因此这里需要使用`git filter-branch`命令来删除`.git`目录中的文件记录：
```bash
$ git filter-branch --index-filter 'git rm -r --cached --ignore-unmatch <file/dir>' -- --all
```
这是在你已知大文件的名字和目录情况下的删除过程。如果过了很久或者是有很多大文件，我们需要有一系列的命令来找出大文件，然后对其进行过滤。下面详细阐述整个过程。
<!--more-->

## 识别出大文件对象
Git中会对大文件进行打包，生成git pack格式的`.pack`文件以及对应的同名的`.idx`文件，存放在`.git/object/pack`目录中。通常来说，Git仓库的大文件都是`.pack`格式的，存放在这个目录中。  

我们可以使用`git verify-pack -v <SHA-1-code>.idx`命令来查看打包文件`*.pack`的内容，如下面是该命令的一个示例输出：
```bash
$ git verify-pack -v .git/objects/pack/pack-2a54c6297dea8fa7feaa30b9738459765bb369a5.idx 
e18ab3d3c2bd2132c65c321dfa8e369756e61326 commit 177 123 12
518876ca5a6f11241e71619a8d677f56863f3e2f blob   6170464 6115358 135
a9ab211dafe06646f182a6f791627f3baf8dd02f tree   44 54 6115493
non delta: 3 objects
.git/objects/pack/pack-2a54c6297dea8fa7feaa30b9738459765bb369a5.pack: ok
```
可以看到这个pack压缩包中有3个文件，对应输出的2-4行，每行的格式如下：
```bash
SHA-1 type size size-in-packfile offset-in-packfile
```
因此我们可以根据每行的第3项的值，即文件的大小对压缩包中的文件进行排序，然后根据大小排序找出大文件。具体的命令如下：
```bash
git verify-pack -v .git/objects/pack/<SHA-1-code>.idx | sort -k 3 -n |tail -n 20
```
上述命令会对对应的压缩文件进行分析，找出其中最大的20个文件。下面是一个示例输出：
```bash
git verify-pack -v .git/objects/pack/pack-318f6bd223ffc6f1cd5675946e9fe7fe11bbaa16.idx | sort -k 3 -n |tail -n 20
18c8efee82a9088820da7b742047a1039e78c2f7 blob   8510 2439 481190
a2b1d917c473e4bbd95d7b35bf01fc16e03e9bfc blob   10780 3355 4354
be6f843c0c6aace758b2657d6c143218b4506544 blob   10923 2158 516005
180ce52e5fc4a1dc2aa304d01a16305bf61c3b1b blob   11186 1223 518163
16ee0f99c0cedd18a93d0bcbf8e37eae2c97d8b4 blob   13025 3549 488959
fac08c493bc1bb4f603d606ac229446a8e0aac04 blob   13595 7857 53828 1 11900fd25b384500e3248576a624e24b67133834
ffe06ba2393aeaba8aa43a634ff0f76394e80aef blob   24063 3100 466731
8203b8c755cc96f3959c79212ae6b45e016e5492 blob   27626 10859 392665 1 ab395d21c693443302aecad6566dd8fb756a40c4
26140b3267d8a81a7e2f3ba72d32dbc63ebd0be3 blob   29333 10427 403524 2 8203b8c755cc96f3959c79212ae6b45e016e5492
222b644716ff079e4a55a151f23d9c98319f1493 blob   29551 12303 454428 1 48db793a769f3d5865a2c6a67519aaeb2cfed8d3
50a2cd63d7e9a38e057943abece20bd32724227d blob   30168 12908 441520 1 48db793a769f3d5865a2c6a67519aaeb2cfed8d3
04423baf437c3ac7252d001f5113322ec5328402 blob   32074 11800 42028 1 11900fd25b384500e3248576a624e24b67133834
9f0eaf7e016a0833a60113270c86db80ed41c385 blob   54208 24360 107858 1 11900fd25b384500e3248576a624e24b67133834
a5fa94cead7a74652e41bf92ad7e9b330d496014 blob   84612 14134 61685
3dbb3684ac8c2009ad2a8e40f24f7475d9a8c1b2 blob   98788 32039 75819 1 11900fd25b384500e3248576a624e24b67133834
ab395d21c693443302aecad6566dd8fb756a40c4 blob   142360 11036 381629
11900fd25b384500e3248576a624e24b67133834 blob   200923 31670 10358
48db793a769f3d5865a2c6a67519aaeb2cfed8d3 blob   398805 27569 413951
bb0319cbbe1760601316c35629009ae2a0ef2fdc blob   460738 148159 233470 1 705d521f9a03ec7ce061653afaf664ab32724dac
705d521f9a03ec7ce061653afaf664ab32724dac blob   1268611 100610 132218
```
其中每行是一个Git的对象（Object)。

## 找出Git对象对应的文件名
由于上述步骤得到的Git对象只有一长串的SHA-1的值，而没有具体的对应的在文件系统中的文件名字，因此我们需要找出Git对象对应的文件名。   
我们可以使用`git rev-list <commit-id>`来达到此功能。 这个命令用来显示某次提交前的所有的提交对象（commit object），而加了`--objects`则用来显示某次提交时所有的Git对象。使用`--all`则显示所有的提交，而不是某次特定的提交下的对象信息。因此用下面的命令可以查看Git对象和对应的文件路径：
```bash
 git rev-list --objects --all |grep <SHA-1-code>
```
以上个步骤中输出的最后一个文件为例，执行这条的命令（注意SHA-1的值只用输入前6位即可）：
```bash
$ git rev-list --objects --all |grep 705d52
705d521f9a03ec7ce061653afaf664ab32724dac data/model-400M.caffemodel
```
可以看出，这个Git对象对应的文件路径是`data/model-4000M.caffemodel`。

## 找出修改这个文件的所有commit
我们需要从commit历史中找到所有修改该文件的commit然后修改这些commit。这里我们使用`git log`来操作，具体如下：
```bash
git log --pretty=online -- <file-name>
``` 
以`data/model-400M.caffemodel`文件为例，这个命令的具体形式为：
```bash
$ git log --pretty=oneline -- data/model-400M.caffemodel
32a9f5f8136c9b011d785bdd08dd24cd0e1d0d1b first commit
```

## 重写所有修改这个文件的提交
找到所有修改这个对象的commit后，我们找到最早的修改，然后使用`git filter-branch`命令来操作，具体如下：
```bash
$ git filter-branch --index-filter 'git rm --cached --ignore-unmatch data/model-400M.caffemodel' -- 32a9f5 
```
也可以将这步和上面一步合在一起，直接从所有提交中删除这个对象：
```bash
$ git filter-branch --index-filter 'git rm --cached --ignore-unmatch data/model-400M.caffemodel' -- --all
```
必要的时候，需要用`-f`选项来强制地进行删除：
```bash
git filter-branch -f --index-filter 'git rm --cached --ignore-unmatch data/model-400M.caffemodel' -- --all
```

## 删除引用并重新打包
这里需要删除`.git/refs`目录下的一些引用文件并重新打包，具体命令如下，比较固定：
```bash
$ rm -Rf .git/refs/original
$ rm -Rf .git/logs
$ git gc
```
之后可以用`du -sh `等命令查看项目目录的大小。  
如果`git push`提示冲突的话，需要用`git push -f`命令来强制推送代码到远端。虽然不建议用`-f`选项，但是特殊情况特殊处理~

## 参考
1. Pro Git 第9.7.3节
2. <https://stackoverflow.com/questions/19573031/cant-push-to-github-because-of-large-file-which-i-already-deleted>

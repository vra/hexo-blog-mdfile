title: Sublime Text 使用技巧1
id: 495
categories:
  - Sublime Text 
date: 2015-05-27 14:10:16
tags:
 - Sublime Text
---

Sublime Text 是一款功能很强大的编辑器，用起来很爽，界面也很华丽。但我看了一系列的学习视频时候，才发现为我对Sublime Text 2的许多功能还是不了解，这里记录下来，记性不好，只能通过别的方法来补充了。下面是一些小技巧。

## 1.打开文件夹并保存为sulime-project

将整个文件夹拖进打开着的Sublime Text 中，然后在工具栏上选择**View->Side Bar->Show Side Bar**，即可看到打开的文件夹了。也可以用快捷键`Ctrl-K,Ctrl-B`来完成该操作。 要将打开的文件夹保存为sublime-project，在工具栏上选择**Project->Save Project As...**然后在打开的对话框中填写保存的项目名，后缀是`sublime-project`。

<!--more-->

## 2.设置首选项

Sublime Text里面有许多的默认选项，如字体大小、tab缩进几个空格等，这些设置都是以类似Json的文本格式保存的。默认的设置文件可以这样打开：工具栏上选择**Preferences->Settings-Defaults**。在Window 7上，这个设置文件是只读的（视频教程里面用的是Mac，可以修改），因此用户可以设置自己的首选项，工具栏上选择**Preferences**->**Settings-User**，设置文件就会打开。建议先读懂默认设置里面的每一项设置的内容（每一项设置的内容都有非常详尽的注释，保证一看就懂），然后再复制到用户设置文件里面修改。

## 3.设置外观

1.  设置配色方案 默认的配色方案有许多，可以通过**Preferences->Color Scheme**来选择。除了默认的方案，还可以通过`Package Control`命令安装喜欢的命令。

2.  设置显示布局 可以将窗口划分为许多小窗口，通过**View->Layout**，选择自己想要的布局，有`Single`，`Columns:2`，`Columns:3`，`Columns:4`，`Rows:2`，`Rows:2`，`Grid:4`几种选项。

## 4.多行选择

多行选择是将多个行选定，然后对这些行一起执行操作，对HTML里面的标签操作很方便。选定多个行的方式是：按住`Ctrl`键，然后在想要操作的行的某个位置点击，即选定该位置。

## 5.插件Emmet的使用

看了介绍，Emmet真是个提高效率的很有用的工具。Emmet利用HTML和CSS代码里面的规范的标签和较多的重复性内容，使用简单的标记方法来简洁地进行代码书写。可以通过`Package Control` 来搜索Emmet来安装。下面简要地介绍下Emmet的一些标记规则,全部规则见[这篇博客](http://www.cnblogs.com/matchless/archive/2013/04/10/3010628.html)。

1.  `#`：代表`id`，例如

    `div#nav`
    效果为
    ```html
	<div id = "nav"></div>
    ```

2.  `.` 代表`class`，例如
    `div.nav
    `
    效果为
    ```html
	<div clas="nav"></div>
    ```

3.  `>`代表包含，即子标签，如
    `div>p>span
    `
    效果为
    ```html
	<div>
     <p><span></span></p>
	</div>
    ```

4.  `+`代表相邻标签，即
    `div>p+span
    `
    代表
    ```html
	<div>
     <p></p>
     <span></span>
	</div>
    ```

5.  `*`代表多个标签，如
    `ul>li*3
    `
    代表
    ```html
	<ul>
     <li></li>
     <li></li>
     <li></li>
	</ul>
    ```

6.  `{}`代表文本内容，如
    `ul>li*3>a{Link}
    `
    代表
    ```html
	<ul>
     <li><a href="">Link</a></li>
     <li><a href="">Link</a></li>
     <li><a href="">Link</a></li>
	</ul>
	```

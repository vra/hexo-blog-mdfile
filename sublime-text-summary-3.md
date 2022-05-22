title: Sublime Text 使用技巧3
date: 2015-06-02 20:47:10
tags:
 - Sublime Text 
---

## 主题管理插件Themr

这个插件用命令的形式来管理、设置主题Theme，省去了点击按钮的繁琐操作，对喜爱简单操作的用户来说很有用。
安装方式：**Package Control Install**->输入**Themr**安装即可。

<!--more-->

## 文件和文件夹的不显示

之前提到过，Sublime Text可以打开一个文件夹，并将文件夹中所有内容列出到左侧。我们可以进行设置，使一些文件夹和文件不显示出来。具体做法如下：

 1. 先将文件夹保存为`sublime-project`: **Project->Save Project As…**，选择保存位置

 2. 重新打开保存的sublime-project文件，就弹出了文件列表和一个配置文件，在配置文件里面添加下面语句：

```javascript
{
	"folders":
	[
		{
			//added part
			"folder_exclude_patterns":["figures"],
			"file_exclude_patterns":["*.md"]
		}
	]
}
```

保存后，可以看到，文件列表里面`figures`文件夹已经不见了，所有`md`格式的文件也步显示了。
注意：设置了不显示后，使用**Ctrl-P**命令搜索内容的时候，被屏蔽的文件夹和文件中的内容是搜不到的。

上面这种方法只能设置当前打开的项目的情况，如果要对所有的工程都屏蔽某一类文件，则可以在**Preferences->Settings-User**中添加上面两条语句，则对所有项目都适用。

## 快速书写CSS代码的插件hayaku

这个插件可以帮助你快速地书写css代码，可以使用简单的几个字母组合就能写出很长的css格式代码，如`ml10`会被解析成`margin-left:10px;`。
安装方法：搜索**hayaku**进行安装即可。

## Sublime Text 3中的代码提示SublimeLinter，注意与Sublime Text 2中很不相同

Sublime Text 3中的代码提示插件**SublimeLinter**改进较大，安装方式也不一样，安装**SublimeLinter**后单独安装针对每一种语言的`linter`，可以先安装**SublimeLinter**，然后看`Readme`文档查看如何安装剩余的部分。
关掉代码提示可以在`Ctrl-Shift-P`搜**SublimeLinter:Toggle** 来设置开启或关闭

## 颜色提示插件Color Hightlight

在编写代码时，颜色的标记常常和颜色对应不上，给出一个颜色标记`#955278`，很难一下子想象到对应的是什么颜色。于是，**Color Hightlight**出现了。安装了这个插件之后，只要点击代码中的颜色标记，就会在该标记上显示对应的颜色，确实很有用的～
安装：搜索**Color Hightlight**安装即可。

## 取色器插件ColorPicker

这个插件可以获取颜色，然后直接在代码中使用。启动插件的快捷键：`Ctrl-Shift-c`。面板出来后一看就知道咋用了。

title: markdown易错点总结
date: 2016-03-29 14:36:25
tags:
- markdown
- 总结
---

markdown 是一种标记语言，我这个博客就是用markdown格式写好后，由hexo框架将markdown格式转换为静态的HTML文件，再上传到网站服务器上。在使用markdown的时候，有的时候在使用有序列表的时候，总会出现一些与预期效果不符的情况。因此今天我查看了markdown的文档，发现有一些规则我之前没注意到，导致出错，所以写下来，避免再犯错了。  
<!--more-->
1.	有序下标中，有多个段落，则下标以及每个段落的开头都必须缩进4个空格或1个制表符
	1.	how are you today?
		I am fine.
	2.	Good Morning!
		Good Morning!
2.	下标中的表示代码段的3个撇号不用缩进
	```cpp
	int main()
	{
		return 0;
	}
	```
3.	不同下标间不用空行，否则也会出错
4.	表示代码结束的三个```后面不能加空格，否则后面的内容也会被当作代码段的。

5. **在GitHub网站上，有序列表的不同下标之间需要隔一个空行，否则渲染会出问题**
参考:<http://wowubuntu.com/markdown/>



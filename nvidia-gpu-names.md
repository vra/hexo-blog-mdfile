title: 关于英伟达显卡命名的姿势
date: 2016-12-18 18:11:11
tags:
 - 总结 
 - Nvidia
 - GPU
---

平时在实验中用到GPU的地方比较多，看新闻也总是能看到英伟达又出了什么型号的显卡等等，可是我一直没搞清楚该公司显卡名称的命名关系，今天特地查了下，总结在这里，以便以后翻阅。  
<!--more-->
Nvidia的GPU命名有4个层次：
 1. GPU 架构(microarchitecture), 表示GPU在芯片设计层面上的不同处理方式，包括的内容有计算单元(SIMD)的个数、有无L1,L2缓存、是否有双精度支持等。按时间顺序依次是Tesla, Fermi, Kepler， Maxwell, Pascal。
 2. 显卡系列：根据使用场景的不同，分成GeForce, Quadro, Tesla。GeForce用于家庭和个人电脑，包括游戏和娱乐等;Quadro用于工业渲染、艺术设计，工作站等场合。而Tesla用于科学计算，深度学习加速等场景。当然这三者的使用场景并没有严格的边界，想GeForce 系列的GTX 1080也可以用来做深度学习实验。
 3. 芯片型号，例如GT200、GK210、GM104、GF104， K80, M40等。其中第二个字母表示架构，如K40 中的K表示是Kepler架构,P100中的P表示Pascal架构。
 4. 针对GeForce系列，还有2系列，3系列，200系列，400系列等分类，像GeForce GTX 1080 就是10系列。
 
需要注意的地方有：
 1. 注意区分Tesla GPU架构和Tesla系列。前者已经用的不是很多了，而后者是最近才出的针对深度学习的系列，使用很多，像我们实验室用的K20,K80都是这个系列。
 2. 描述一个显卡的时候，一般是系列名+芯片型号，如 Tesla K80。 
 3. 针对GeForce系列，芯片型号一般是显卡型号+具体编号的形式，如 GeForce GT 705,其中GT 是显卡型号。
 4. 最近新出了一款 TiTan X, 主要要和GeForce GTX Tian X 区分。

## 参考：
 1. <https://chenrudan.github.io/blog/2015/12/20/introductionofgpuhardware.html>
 2. <https://www.quora.com/What-is-NVIDIA-GPU-micro-architecture>
 3. <https://en.wikipedia.org/wiki/List_of_Nvidia_graphics_processing_units>

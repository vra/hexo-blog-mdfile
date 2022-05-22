title: 如何用matlab画稍微美观点的图
date: 2015-05-31 20:59:42
tags:
 - 学习总结
 - matlab
---

本科毕设论文写作过程中，老师指出我用matlab画的图太丑，需要好好改改。于是我这几天参考网上资料，对画图的一些细节进行了设置，得到的图确实比以前好了些。而且我matlab用的不多，很多东西这次用过，下次碰可能要过很长时间，许多之前记得的东西都忘了，所以写下来是很有必要的。另外我现在画的图也只是比之前稍微好点，所以就起了这样一个题目。

<!--more-->


## 1. 设置plot

参考内容：<http://www.mathworks.com/help/matlab/ref/plot.html>

### 设置曲线形式(LineSpec)

曲线形式包括3个部分，分别是`Line Style`，`marker symbol`和`color`。

1. `Line style`表示曲线的类别，有4类：

 1. -：实线(Solid line)，也是默认的格式
 2. --:虚线(dashed line)，也就是------这种格式
 3. ::点线(dotted line),也就是将引号:横过来的格式
 4. -.:点虚线(Dash-dot line),就是虚线和点交替出现,-.-.-.-.,不过点是在中间，跟虚线相平的。

2. marker symbol表示数据点的标记形式，有如下几类，直接复制过来了：

 1. o Circle
 2. + Plus sign
 3. * Asterisk
 4. . Point
 5. x Cross
 6. s Square
 7. d Diamond
 8. ^ Upward-pointing triangle
 9. v Downward-pointing triangle
 10. `>` Right-pointing triangle
 11. < Left-pointing triangle
 12. p Pentagram
 13. h Hexagram

3. color表示颜色设置，matlab画图里面所有的颜色都是这几种：
 
 1. y yellow
 2. m magenta,品红
 3. c cyan,青色
 4. r red
 5. g green
 6. b blue
 7. w white
 8. k black

需要主要的有2点：

 1. 这三个选项可以省略任意一个或多个，当省略`line style`且设定了`marker symbol`时，这时候得到的只有数据点，没有曲线。
 2. 如果Y值是一个矩阵的时候，如果设定了`color`选项，怎对所有曲线，颜色都是设定的那种颜色；如果没设定`color`,则曲线颜色按上面所示颜色顺序依次往下选择。

### 设置曲线宽度:LineWidth

曲线宽度设置好也是很重要的，默认的曲线太细，不美观，我们可以使用LineWidth来设置，其单位为点的大小,比如

```matlab
plot(x,y,'LineWidth',2)
```

表示线宽是两倍的点大小。

### 设置标记大小:MarkerSize

标记大小表示前面设定的`marker symbol`的大小，单位为点的大小。

### 设置标记边缘颜色:MarkerEdgeColor

标记边缘颜色就是标记周围一圈的颜色。

### 标记填充颜色:MarkerFaceColor

标记填充颜色。


## 2. 设置网格Grid

参考内容：

1. <http://www.mathworks.com/help/matlab/ref/gca.html>
2. <http://www.mathworks.com/help/matlab/ref/grid.html>
3. <http://www.mathworks.com/help/matlab/ref/axes-properties.html#prop_MinorGridLineStyle>
4. <http://www.mathworks.com/help/matlab/ref/axes-properties.html#prop_GridAlpha>

### 显示/隐藏网格

1. grid on：显示网格
2. grid off：隐藏网格

### 得到当前坐标轴Axis

`ax = gca;`:得到当前坐标轴，其中`gca`意为get current axis，实际是一个函数，只不过后面没加括号而已。
得到坐标轴后，就可以对图像进行一系列的设置。

### 设置网格线类型:GridLineStyle

```matlab
ax= gca;
ax.GridLineStyle = ':'; %设置网格线为点线
```

### 设置网格线透明度：GridAlpha

默认透明度是0.15，可以使用GridAlpha来设置

```matlab
ax = gca;
ax.GridAlpha = 0.5 %设置透明度为0.5
```

## 3. 设置坐标轴

### 设置坐标轴范围

可以用`axis([xStart xEnd yStart yEnd])`这样一条命令来设置坐标轴的范围。

```matlab
axis([0.1 0.6 0.5 0.8]);%x轴从0.1到0.6，y轴从0.5到0.8
```

### 设置坐标轴跨度

```matlab
set(gca, 'xtick',[xStart:xStep:xEnd]):设置x轴步长，从xStart开始，从xEnd结束，步长是xStep。
set(gca, 'ytick',[yStart:yStep:yEnd]):同X轴。
```

## 4. 设置图标题和图中文字

参考内容：

<http://www.mathworks.com/help/matlab/ref/legend.html>

### 设置图片标题：legend

```matlab
legend('figure1');
```

### 设置x/y轴坐标：x/ylabel

```matlab
xlabel('\alpha');
ylabel('soccer');
```

### 设置图中文字：text

使用方法：text(xPos, yPos,'str')

```matlab
for i = 1 : 10
	text(x(i)+0.1, y(i)+0.3, num2str(y(i))); %num2str：将数字转换为字符
end
```

上面代码表示在(x+0.1, y+0.3)处显示y的值

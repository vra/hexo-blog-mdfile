title: cool-certificate, 一个好玩的证书生成工具
date: 2016-04-07 23:15:03
tags:
- Python
- 图像处理
- 工具
- PIL
---

前几天同学发过来一张无人机驾驶证的照片，瞬间觉得很高大上，仔细一询问，原来是用软件生成的图片，网址是：<http://wx.znl.cn/app/index.php?i=120&c=entry&id=1&do=index&m=bi_pic>。 当访问该网站的时候，用户输入用户名，然后就生成包含用户名的驾照照片。我接着想能不能自己做一个类似这样的东西呢，经过思考发现，其实操作比较简单，即将用户姓名写入到图像上的合适位置即可。因为我之前已经有一些用Python 的Django框架做小的网站的经验，而且Python PIL模块可以完成这个任务，所以我立即想到， 能不能结合两者，建立一个网站，让用户输入姓名，然后将用户姓名传入到后台，后台调用PIL函数，将名字写到图片的相应位置上，然后返回给用户呢？经过思考我发现这种思路是可行的，而且工作量貌似也不是很大，所以今天早上开始做了做，在无人机驾照的基础上又增加了2个有趣的证件：潜水证和超级帅哥证，今晚终于作出了一个粗糙的结果（网站页面使用了原始和简单的HTML标签），可以在[这里](http://115.28.30.25:8001/)访问。代码已经上传到[github上](https://github.com/vra/cool_certificate)了。下面记下来实现过程中的一些思考。
<!--more-->

## 整体实现流程
1.	用Django实现网站前端和后端，展示页面给用户，读取用户输入
2.	当用户输入后，利用POST方法返回用户名到服务器端
3.	对特定的证件和已给的用户，利用PIL中的ImageFont模块来在证件照片的相应用户名空当处写上用户名,然后保存处理后的图片。用户名应该写在哪里需要手工确定（我用Windows 的画图工具中找到具体的位置坐标）
4.	将生成的图片返回给网站页面

## 实现的一些细节问题
### 将文字写到图片上
这里使用PIL（Python Image Library）来做，利用了其中的ImageFont模块，核心的代码段如下：
```python
	img = Image.open(img_path)                                                                                                                                                   draw = ImageDraw.Draw(img)                                                                                                                                               
    font = ImageFont.truetype(font_path, font_size)                                                                                                                          
    #NOTE: django get parameter as unicode, so don't need to encode to unicode.                                                                                                  draw.text(word_pos, name, word_color, font=font)                                                                                                                         
    img.save(out_img_path)                                       
```
One things you must notice is that Django return stirng in the unicode format, so you dont't have to do `unicode(name, 'utf-8')` anymore.  
用户输入姓名时，生成包含姓名的证件图片，保存在本地。  
在实际操作中发现，有些字体不支持部分中文，所以我在网上下了`Aria Unicode`字体，经测试发现能显示所有中文字体。  

### Django返回处理图片的格式
我最初想的是用户点击确定按钮后，跳转到新的页面，在这个页面上单独显示处理后的照片，所以response类型设置成`image/jpeg`即可。但实际操作中出现问题，只返回照片似乎有一些问题，所以我修改实现，在传给Template的时候，传递一个参数`done`, 如果当前没有增加用户姓名，则该值为0,否则为1。在Template中，如果值为0,则展示未处理的模板图片;如果值为1,则显示处理后的图片。


### 静态文件目录的设置
Django将CSS,JS和Image图片都看作静态文件，推荐在app目录下建立`static`目录来保存这些文件。这里需要进行一定的设置，将保存模板图片和生成图片的目录`imgs`增加到`static`目录下，设置代码如下:
```python
# in settings.py
SITE_ROOT = os.path.join(os.path.abspath(os.path.dirname(__file__)),'')                                                                                                      
STATIC_ROOT = os.path.join(SITE_ROOT,'static')                                                                                                                               
STATIC_URL = '/static/'                                                                                                                                                      
                                                                                                                                                                             
#最后关键部分需要添加上STATICFILE_DIRS的配置                                                                                                                                 
STATICFILES_DIRS = (                                                                                                                                                         
    ("css", os.path.join(STATIC_ROOT,'css')),                                                                                                                                
    ("js", os.path.join(STATIC_ROOT,'js')),                                                                                                                                  
    ("imgs", os.path.join(STATIC_ROOT,'imgs')),                                                                                                                              
)                                              
```
经过这样设置，在调用`imgs`目录下的图片时就可以这样调用了：
```python
	<img src = "{% static "imgs/feiji.jpg"%}"> 
```
	

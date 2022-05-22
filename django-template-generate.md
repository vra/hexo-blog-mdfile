title: 将现有的Web前端项目生成导入到Django的Template
date: 2015-10-10 22:43:43
tags:

---

实际项目中，会遇到这样的问题：没有使用任何服务器端框架的前端代码，即包含html网页文件，也包含js和css的代码，如何将这些现有的项目做最少的修改而引入到Django框架中呢？Django官网上给出了解决方法，使用`static`目录来存放`css`和`js`代码（虽然`js`是动态代码，但Django将其与`css`等同为`静态`代码，因为在后端看来，前端代码是静态的），然后在`html`文件里面，将原先的`href`引用改为通过`static`目录来引用。可以看[这里](https://docs.djangoproject.com/en/1.8/howto/static-files/)，但里面讲的不是很清楚，我在查了一些资料后才搞定这个问题，所以这里写个总结来总结总结。  

<!--more-->

## 修改配置文件，增加`static`相关目录
在配置文件`settings.py`里面，增加`STATIC_ROOT`，`STATIC_URL`和`STATICFILES_DIRS`变量，使得程序在执行时知道从哪里读取配置文件：  

```py
SITE_ROOT = os.path.join(os.path.abspath(os.path.dirname(__file__)),'')
STAIC_ROOT = os.path.join(SITE_ROOT,'static')
STATIC_URL = '/static/'

#最后关键部分需要添加上STATICFILE_DIRS的配置
STATICFILES_DIRS = (
    ("css", os.path.join(STATIC_ROOT,'css')),
    ("js", os.path.join(STATIC_ROOT,'js')),
    ("images", os.path.join(STATIC_ROOT,'images')),
)
```
上面代码中，为了更容易地表示`STATIC_ROOT`的值，先获取了`SITE_ROOT`的值。 
注意：这个设置只能在`DEBUG=True`，即处于开发状态的的时候才有用，实际生产环境中的配置还有些区别。  

## 在app里面创建`static`目录
在相应的app里面创建好`static`目录，然后将现有项目的`css`和`js`目录拷贝到该目录下。 至于`html`文件，则放在相应的`templates`目录下。 

## 修改`html`文件里面的`href`引用
因为原先项目中，对于`Javascript`和`CSS`代码的引用都是通过相对目录来引用的，例如：  
```html
<link rel="stylesheet" type="text/css" href="../css/bootstrap.css">
<link rel="stylesheet" type="text/css" href="../css/jquery.fullPage.css">
```
而在Django里面，需要对相对目录进行修改，将其改为通过`static`来引用的方式，也很简单：  
```html
{% raw %}
** {% load staticfiles %}**
{% endraw %}
<link rel="stylesheet" type="text/css" {% raw %} href="{% static "css/bootstrap.css" %}" {% endraw %}>
<link rel="stylesheet" type="text/css" {% raw %} href="{% static "css/jquery.fullPage.css" %}"> {% endraw %}
```

我们可以看到主要有2处修改：
 1.增加了 {% raw %}`{% load staticfiles %}` {% endraw %}语句，其中staticfiles是Django自带的库，{% raw %}`{% %}`{% endraw %} 是Django的模板语法。这条语句表示导入staticfiles模块。  
 2. 将href中的引用修改为{% raw %} `href="{% static "subfolder/filename" %}"` {% endraw %}的格式，也很好理解，相当于文件引用路径是`static` + `subfolder/filename`，即通过前面`settings.py`里面设置的`static`目录来寻找`css`和`js`文件。  

## 页面跳转的问题
还遇到了一些问题，比如说在现成的前端项目中，我们要跳转到别的网页，我们可以这样写：   
```html
<a href="something.html">Something</a>
```
但在Django里面，却要改为：  
```html
<a href="/something/">Something</a>
```
否则会跳转出错。



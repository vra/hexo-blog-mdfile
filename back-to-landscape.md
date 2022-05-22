---
title: back-to-landscape——博客迁移记录2021
date: 2021-09-04 23:46:21
tags:
 - 迁移记录
 - Hexo
 - 总结
---
## 概述
2019年的时候，写了一篇[博客](https://vra.github.io/2019/02/27/mv-to-next/)来记录博客历史的迁移记录，这两年又经过工作变化、硬盘损坏，博客也是几经变迁。

尝试了基于Go的hugo框架，总体美观度和Hexo还是没法比，因此还是切换回了Hexo，换用了默认的landscape主题，重心放到有效的内容的记录上。评论系统还是采用valine，而在landscape下，设置valine还比Next复杂一些，我从[这里](http://hypo1986.com/blog/2019/06/10/hexo-landscape-add-valine/) 看到除了配置landscape项目，还需要在ejs文件里面设置，这里记录下。
<!--more-->

## 详细流程
修改主题config 文件 `HEXO_ROOT/themes/landscape/_config.yml`, 添加下面内容:
```yaml
# valine comment system. https://valine.js.org
valine:
  enable: true # if you want use valine,please set this value is true
  appid: wwwwweirowjreojwreoz # leancloud application app id
  appkey: weiojwoerjoerj# leancloud application app key
  notify: false # valine mail notify (true/false) https://github.com/xCss/Valine/wiki
  verify: false # valine verify code (true/false)
  pageSize: 10 # comment list page size
  avatar: mm # gravatar style https://valine.js.org/#/avatar
  lang: zh-cn # i18n: zh-cn/en
  placeholder: 欢迎留言交流~~ # valine comment input placeholder(like: Please leave your footprints )
  guest_info: nick,mail,link #valine comment header info
```
appid 和 appkey 从 leancloud 网站获取.


修改ejs文件`HEXO_ROOT/themes/landscape/layout/_partial/after-footer.ejs`，添加下面内容：
```javascript
<% if(theme.valine.enable && theme.valine.appid && theme.valine.appkey){ %>
  <script src="//cdn1.lncld.net/static/js/3.0.4/av-min.js"></script>
  <script src="//unpkg.com/valine/dist/Valine.min.js"></script>
  <script>
    var GUEST_INFO = ['nick','mail','link'];
    var guest_info = '<%= theme.valine.guest_info %>'.split(',').filter(function(item){
        return GUEST_INFO.indexOf(item) > -1
    });
    var notify = '<%= theme.valine.notify %>' == true;
    var verify = '<%= theme.valine.verify %>' == true;
    new Valine({
      el: '.vcomment',
      notify: notify,
      verify: verify,
      appId: "<%= theme.valine.appid %>",
      appKey: "<%= theme.valine.appkey %>",
      placeholder: "<%= theme.valine.placeholder %>",
      pageSize: '<%= theme.valine.pageSize %>',
      avatar: '<%= theme.valine.avatar %>',
      lang: '<%= theme.valine.lang %>',
      visitor: 'true'
    });
  </script>
<% } %>
```

修改`HEXO_ROOT/themes/landscape/layout/_partial/article.ejs` 文件，最后添加下面内容：
```javascript
<% if (!index && post.comments && theme.valine.enable && theme.valine.appid && theme.valine.appkey){ %>
  <section id="comments" class="vcomment">
  </section>
<% } %>
```





title: 初探Grunt
date: 2015-07-04 20:17:57
tags:
 - Web编程
---
最近打算学习一些web编程的知识，今天学习了Grunt这个工具的用法，这里简要地对学习的知识点进行个总结。

##1. Grunt是什么

Grunt网站上的副标题是”The Javascript Task Runner”，是用来实现Javascript编程自动化的一个工具，类似`make`工具体系。只要设置好`Gruntfile`（类比`Makefile`），就可以使用`grunt`命令来自动执行javascript代码的清理、重新生成等任务。Grunt生态圈里面有大量的插件，Grunt工具就是使用这些插件来实现自动化。

<!--more-->

##2. 如何安装Grunt

Grunt通过`npm`命令来安装，所以需要首先安装npm。npm是nodejs package manager的缩写，是nodejs的包管理工具。在新版的nodejs里面默认包含了npm，所以只需要安装最新班的nodejs即可，访问nodejs官方网站下载最新版的nodejs。
之后通过npm安装`grunt-cli`，即Grunt command line interface。为了在所有目录下都可以使用grunt命令，需要加-g参数，指令如下：

```bash
npm install -g grunt-cli
```

注意：有的发行版在使用`npm`命令时需要root权限，前面要加`sudo`命令。
其实安装完grunt-cli后，并没有安装grunt。这里面的原理大概是这样的：grunt-cli只用来寻找通过nodejs的`require`工具(或在package.json的dependencies)已经安装好的本地的grunt,然后执行之。可以看源代码查看工作原理。


##3. 使用Grunt工具前需要准备哪些东西

按理来说，使用`grunt`命令，只需要有个`Gruntfile`就可以了，但是上文提到，grunt task runner需要在每个项目中单独安装，所以还得有个保存项目元数据的`package.json`文件。

在每个nodejs项目中，都有个`package.json`文件来保存这个项目的名称、版本、依赖库等元数据。

`package.json`可以使用命令`npm init`交互式地生成。在生成该文件后，可以使用`npm install`在当前项目目录下安装依赖库。

此外，在项目目录下安装工具库并使用`--save-dev`或`--save`参数，可以将安装的工具自动加入到该项目的依赖库中。其中`--save`命令将安装的工具名称和版本号加入到`dependencie`部分，`--save-dev`则加到`devDependencies`部分。如下命令：

```bash
npm install grunt --save-dev
```

将安装grunt task runner 并将其名称和版本号自动加入到`devDependencies`部分。

`Gruntfile`是`Gruntfile.js`(Javascript语言格式)和`Gruntfile.coffee`(CoffeeScript格式)之一,类似Make工具体系中的`Makefile`，用来保存配置信息，是Grunt工具的最主要文件。
下面是一个Gruntfile的示例格式，详细格式和说明请参阅官方文档。

```javascript
module.exports = function(grunt) {

  grunt.initConfig({
    jshint: {
      files: ['Gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
      options: {
        globals: {
          jQuery: true
        }
      }
    },
    watch: {
      files: ['<%= jshint.files %>'],
      tasks: ['jshint']
    }
  });

  grunt.loadNpmTasks('grunt-contrib-jshint');
  grunt.loadNpmTasks('grunt-contrib-watch');

  grunt.registerTask('default', ['jshint']);

};
```

##4. 如何运行Grunt

在`Gruntfile`写好之后，运行`grunt`命令，就会自动执行`Gruntfile`里面的语句了。so easy 是不是～



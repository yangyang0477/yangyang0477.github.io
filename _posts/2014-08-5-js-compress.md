---
title: JS代码混淆和压缩
layout: post
guid: 7
tags:
  - JS
   
---


##**JS代码混淆**
最近实习参与的项目2.0版本要发布了，由于整个项目是基于前端脚本语言JS编写的，为了让一般用户不那么轻易的获得源代码，保护一下知识产权，同时压缩一下代码，减少浏览器加载时间。做一些必要的代码混淆和加密是必不可少的，当然对于高级用户来说，由于JS代码浏览器前端解析执行的特殊性，得到源码也是根本没有办法阻止的……

最终项目使用了[UglifyJS](https://github.com/mishoo/UglifyJS2)这个开源工具，来对JS代码进行简单的混淆和压缩，UglifyJS需要运行在Node环境下的，所以在安装之前，我们必须先配置好本地的Node环境([node下载地址](http://nodejs.org/))。然后通过自带的NPM来安装UglifyJS，可参考如下两篇文章：

- [用UglifyJS2合并压缩混淆JS代码](http://blog.fens.me/nodejs-uglifyjs2-js/?utm_source=tuicool)
- [小tip：我是如何初体验uglifyjs压缩JS的](http://www.zhangxinxu.com/wordpress/2013/01/uglifyjs-compress-js/)

上面两篇文章基本上已经涵盖了UglifyJS的基本使用，使用过程中，再通过UglifyJS其官方文档，就可以很容易的进行JS代码的混淆和压缩了。

由于一个项目一般会由很多JS组成，一个一个去处理这些JS文件的话，是相当麻烦且不可取的。对此，我们需要写一个批处理的bat文件，来处理一个文件夹目录下的多个JS文件，代码如下：

    @echo off
    SET JSFOLDER=D:\yangyang\uglify-js\diagram
    echo 正在查找JS文件
    chdir /d %JSFOLDER%
    for /r . %%a in (*.min.js) do (
    del %%~fa
    @echo 正在删除 %%~fa
    pause
    )
    
    for /r . %%a in (*.js) do (
    uglifyjs %%~fa -m -r 'require' -o %%~na.min.js
    @echo 正在压缩 %%a
    )
    
    echo 完成!
    pause
    
项目如果使用了AMD方式来加载JS文件，比如使用了RequireJS那么需要对require这些特殊变量进行保留，比如我们使用了jQuery那么对变量$也需要保留，如果这些变量被混淆了，会导致混淆压缩后的代码无法正常运行。上述代码中的-r就是为了保持require这个变量不被混淆。


##**JS代码Eval压缩**
在对JS代码进行简单的混淆和压缩之后，我们可以进一步的对JS代码进行JS Eval方式的压缩，让想得到我们源代码的童鞋，再多费点步骤……
比如我们想对如下JS代码进行Eval压缩：

    alert(“hello world”)
    
Eval压缩之后，我们会得到这样的代码：

    eval(function(p,a,c,k,e,r){e=String;if('0'.replace(0,e)==0){while(c--)r[e(c)]=k[c];k=[function(e){return r[e]||e}];e=function(){return'^$'};c=1};while(c--)if(k[c])p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c]);return p}('alert(“hello world”)',[],1,''.split('|'),0,{}))
    
其实不能算是压缩了……只是让代码变得复杂点。

这里主要使用了[CSDN上面的这个工具](http://download.csdn.net/detail/vni2007/7319903)，这是一个网页小工具，需要丢到Tomcat中进行使用，下面简单介绍一下使用方式:

 1. 解压工程
 2. 将整个工程文件丢入tomcat webapp目录
 3. 启动tomcat服务
 4. 浏览器打开本地地址 http://localhost:8080/compressor/


如图所示:界面非常简洁，支持单文件处理和单个目录处理两种方式，将我们刚才经过UglifyJS压缩和混淆的目录填入相应位置，然后**走起**~

![eval压缩](/media/files/2014/08/05/2.jpg)

**注**:也许在压缩过程中，你会进行对一个目录下的多个文件进行更改名称的操作，介绍大家一个很好用的小工具[超级文件批量重命名工具v1.0](http://www.downxia.com/downinfo/42695.html)，真的很好很强大。

![小工具](/media/files/2014/08/05/3.jpg)

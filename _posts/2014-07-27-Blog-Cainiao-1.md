---
title: 如何给博客内容添加目录索引（菜鸟建博客系列）
layout: post
guid: 4
tags:
  - Blog
  - Github
  - Jekyll
  - 菜鸟建博客
   
---




![目录](/media/files/2014/07/27/1.jpg)

## **博客**

在一周的修修补补中，博客的功能日渐完善，余下就差书写了。计划一周最少写两篇博客，无论是蹩脚技术的分享还是生活琐碎的记录，希望自己能一直坚持下去，培养写作的习惯。下两周准备在实习之余，每晚抽空写一些内容，争取能写一个Github+Jekyll菜鸟建博客系列。
  
## **目录设计**
对于目录我们应该都非常的熟悉，平时阅读的书籍、PPT、WORD、PDF文档都会看到目录的存在。在大量的文本信息中，目录的存在为信息的检索带来了便捷，方便阅读。对于博客来讲，对目录的需求其实并没有书籍那么强烈，因为大部分的博客内容也不会太长，且大部分的浏览器都可以Ctrl+F来检索内容。但是为了让读者在阅读博客内容之前，对博客内容有一个大致的预估，提供一个目录功能，在提升用户体验方面还是不错的。（理由是不是太牵强了……）

好吧既然决定给博客添加目录的功能，那么我们先设计一下博客的目录应该具有什么样的功能：

1. 网页加载完成后，自动根据博客内容，提取**\<h2>\<h3>**标签内容，作为目录Content
2. 目录生成后在网页右侧展现,随着页面的滚动而滚动
3. 点击目录内容，页面自动滚动到相应章节

其实对于博客目录的功能上面三条是比较基础的，我们可以打开百度百科，看一下[百科页面](http://baike.baidu.com/subview/4055/4055.htm#2)所设计的目录功能。o(︶︿︶)o 唉，我这蹩脚的技术还是有差距的，待哪天有空好好的模仿学习一下。

![百度百科目录](/media/files/2014/07/27/2.jpg)

## **开始编码**
由于目录功能的实现中，会对网页DOM结构进行操作，所以本例子中使用[JQuery](http://jquery.com/)来操作DOM，代码部分参考了[http://www.pchou.info/](http://www.pchou.info/)博客中目录功能的实现。

### 网页加载完成？

如何判断网页已经加载完成了呢？JQuery中有$(document).ready()，传统JavaScript有window.onload方法，不过与这两种方法还是有区别的。[区别~](http://www.jb51.net/article/21628.htm) 为了方便阅读我这里就直接粘贴过来了。

**1.执行时间**

> window.onload必须等到页面内包括图片的所有元素加载完毕后才能执行。$(document).ready()是DOM结构绘制完毕后就执行，不必等到加载完毕。

**2.编写个数不同**

> window.onload不能同时编写多个，如果有多个window.onload方法，只会执行一个 $(document).ready()可以同时编写多个，并且都可以得到执行 

**3.简化写法**

> window.onload没有简化写法，$(document).ready(function(){})可以简写成$(function(){});

### 获取h2,h3标签内容
网页加载完成之后，我们首先需要获取DOM中h2,h3标签对象，分配ID，方便后续操作。


    function initHeading(){
        var h2 = [];
        var h3 = [];
        var h2index = 0;
        //获取DOM中h2,h3标签
        $.each($('h2,h3'),function(index,item){
            if(item.tagName.toLowerCase() == 'h2'){
                var h2item = {};
                h2item.name = $(item).text();
                h2item.id = 'menuIndex'+index;
                h2.push(h2item);
                h2index++;
            }else{
                var h3item = {};
                h3item.name = $(item).text();
                h3item.id = 'menuIndex'+index;
				if(h2index-1<0){
					alert('2货别在使用h2标签之前使用h3标签');
				}
                if(!h3[h2index-1]){
                    h3[h2index-1] = [];
                }
                h3[h2index-1].push(h3item);
            }
            item.id = 'menuIndex' + index;
        });
        return {h2:h2,h3:h3}
    }

### 拼接UL标签的str字符串

获取原始博客h2,h3标签内容之后，需要重新组装标签内容，再以<ul>标签作为容器，将这些内容组装起来


    function genTmpl(){
        //开始拼接
        var tmpl = '<ul><li style="list-style-type:none;"><h2>目录<h2></li>';
        var heading = initHeading();
        var h2 = heading.h2;
        var h3 = heading.h3;
        for(var i=0;i<h2.length;i++){
            tmpl += '<li><a href="#" data-id="'+h2[i].id+'">'+h2[i].name+'</a></li>';
            if(h3[i]){
                for(var j=0;j<h3[i].length;j++){
                    tmpl += '<li class="h3"><a href="#" data-id="'+h3[i][j].id+'">'+h3[i][j].name+'</a></li>';
                }
            }
        }
        tmpl += '</ul>';
        return tmpl;
    }
    
### 加入DOM结构中同时注册点击事件

当点击目录内容时，我们期望页面滚动到相应章节，为了有动画效果，采用JQuery中animate()方法，首先变量scrollNum记录每个章节在博客中的高度位置(这时候分配给h2,h3的ID就有用了~)，然后滚动到相应页面高度即可。

    function getIndex(){
                var tmpl = genTmpl();
                //创建div标签
                var indexCon = '<div id="menuIndex" class="sidenav"></div>';
                //加载到页面中
                $('#content').append(indexCon);
                $('#menuIndex')
                    .append($(tmpl))
                    .delegate('a','click',function(e){
                        var selector = $(this).attr('data-id') ? '#'+$(this).attr('data-id') : 'h1'
                        var scrollNum = $(selector).offset().top;
    					//console.log(scrollNum);
                        $('body, html').animate({ scrollTop: scrollNum - 15 }, 300, 'linear');
                    });
            }

基本上功能已经完成，接下来给#menuIndex\<div>标签添加一些css样式就好

    #menuIndex {
      position: fixed;
      margin-left: 750px;
    }
    #menuIndex ol, ul {
      list-style-position: outside; }
    
    #menuIndex .h3{
      font-size:0.8em;
      margin-left:10px;
    }
    #menuIndex .h2{
      font-size:1.2em;
    }	
    #menuIndex a:hover {
      color: #dd1144;
    }
    	
## **效果展示** 
   
大功告成~效果如下图所示。js代码[地址](/media/js/sidenav.js)，css样式[地址](/media/css/style.css)。
![效果图](/media/files/2014/07/27/3.jpg)


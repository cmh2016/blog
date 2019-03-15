---
title: flex--弹性布局--布局的革命
date: 2016-03-23 16:26:57
tags: [flex,css3]
categories: webapp
---
基本概念
=======
* CSS3为我们提供了一种可伸缩的灵活的web页面布局方式-flexbox布局，它具有很强大的功能，可以很轻松实现很多复杂布局。然而Flex属性较多，不便于记忆，而在项目中，布局使用频繁，那么可以将flex抽离为一个布局工具类，简化使用方式，快速应用于项目，减少记忆成本。

* 采用Flex布局的元素为，称为 Flex容器 ，容器的直接子元素称为 Flex项目 ，容器默认有两个轴心线，用于项目的对齐与排列，水平主轴为 main axis ，垂直主轴为 cross axis ，主轴的开始位置为 start ， 结束位置为 end 。

![](http://img2.tuicool.com/nuI77n2.png!web)
CSS属性说明
===========
* 将任意元素的 display 属性设置为 flex ，可将其转换为Flex容器
> 注意：设为Flex容器后，子元素的 float 、 clear 和 vertical-align 属性将失效

快速上手
======
> 感谢lzxb的封装，使用flex.css，简洁的api，熟悉的属性值，入门毫无压力。在html中采用属性的方式布，将布局和css进行分离，清晰的布局结构让你更容易维护，可以在不更改css的情况下更改布局。特别是在React中使用data-flex属性布局，维护起来更加的方便。

使用示例
========
```
<footer flex="dir:left box:mean">
    <a href="index.html" flex="dir:top box:mean">
        <span class="icon icon_index icon_index_active"></span>
        <span class="name active">首页</span>
    </a>
    <a href="classify.html" flex="dir:top box:mean">
        <span class="icon icon_classify"></span>
        <span class="name">分类</span>
    </a>
    <a href="my.html" flex="dir:top box:mean">
        <span class="icon icon_my"></span>
        <span class="name">我的</span>
    </a>
</footer>
```

![](http://ol1kqeyve.bkt.clouddn.com/17-2-8/28559808-file_1486543581319_717c.gif)

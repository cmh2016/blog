---
title: fis3-目前配置最简单的前端工程构建工具
date: 2016-10-16 14:53:57
tags: [fis3,前端自动化,前端性能优化,前端工具]
categories: 前端工具
---
是什么？
=======
>借用官网图片，一看就懂  [官网传送门](http://fis.baidu.com/)

![](http://ol1kqeyve.bkt.clouddn.com/17-2-10/65912440-file_1486711899137_d94.png)

* 本人是今年年初开始在项目中使用fis3来进行前端自动化，模块化，性能优化等工作。从刚开始的fis2使用到fis3的变革，百度其实还是蛮用心的。无论是设计理念和使用体验上都翻了翻，不但具备了Webpack等工具的所有功能，还有许多理念都属前沿，是我最喜欢的前端工程化工具 。

* 说起前端工程化工具就不得不提目前最火的 webpack 、Grunt 、Gulp等。他们的共同点就是让前端攻城狮从重复的工作中解脱出来，更加高效和高质量的完成工作。

* FIS3 , 为你定制的前端工程构建工具。解决前端开发中自动化工具、性能优化、模块化框架、开发规范、代码部署、开发流程等问题

* webpack貌似是目前使用最为火热的打包工具，各大知名的框架类库都用其打包，国内使用最近也火热起来。它在单页应用和类库打包上帮助许多人从代码管理中解脱了出来，成为了当下风靡一时的打包工具。特别是在webapp单页应用快速发展的今天，vue.js凭借着易用、灵活、高性能的特点脱颖而出。官方提供的vue全家桶(vue+vueRouter+webpack)着实让webpack火了一把。

    配置文件
    ======
    >`fis-conf.js`

    ```
    /*
    * @Author: cmh
    * @Date:   2016-01-13 09:56:40
    * @Last Modified by:  cmh
    * @Last Modified time: 2016-07-07 13:57:22
    */

    //打包
    fis.match('::package', {
    postpackager:fis.plugin('loader', {
    allInOne: true
    })
    });
    //开启MD5戳 项目完成时统一加md5
    fis.match('*.{less,js,css,png,jpg}', {
    useHash: true
    });
    // widget 目录下为组件
    fis.match('/widget/**', {
      isMod: true
    });
    // widget下的 js 调用 jswrapper 进行自动化组件化封装
    fis.match('/widget/*.js', {
        postprocessor: fis.plugin('jswrapper', {
            type: 'commonjs'
        })
    });
    //编译
    fis.match('*.less', {
    parser: fis.plugin('less'),
    rExt: '.css'
    });

    //压缩
    fis.match('*.js', {
    optimizer: fis.plugin('uglify-js')
    });
    fis.match('*.less', {
    optimizer: fis.plugin('clean-css')
    });
    fis.match('*.css', {
    optimizer: fis.plugin('clean-css')
    });
    fis.match('*.png', {
    optimizer: fis.plugin('png-compressor')
    });
    // 启用相对路径插件
    fis.hook('relative');
    // 让所有文件，都使用相对路径。
    fis.match('**', {
    relative: true
    });

    fis.match('*.{less,css,js,png,jpg,ttf}', {
      release: '/static/$0' // 所有静态资源发布时产出到static下
    });
    fis.match('widget/**.html', {
      release: '/template/$0' // 所有模板资源发布时产出到static/template下
    });
    ```
    我的目录结构
    =========
    ```
    --------- css/项目用到的css文件
|
--------fonts/项目中使用的字体文件
|            
|            -----icon/项目用到的所有图标文件
|            |
------images------banner/项目banner图
|            |
|            -----test/项目静态资源的示例图片
|
|            ------lib/项目用到的js库文件
-----static---------js----|
|            ------my/项目用到的js文件
|
--------pkg/项目页面中引用的打包好的单页面css/js文件
|
|
--------template/widget/项目中模块化的html/js文件
|
|
---------map.json/项目资源地图
|
|
---------*.html/项目最终生成的静态页面
|
|
----------目录结构说明.txt
```

项目实战
======

>html模块化，可以方便的在需要的位置引入公用的html结构（类似于php的include("xxx.php")

举个例子，比如客户说我要改一个头部菜单的二级菜单，以往我们会修改一个，然后复制粘贴.....。现在只需要修改头部模块，剩下的交给fis3来帮你完成。

>比如list页面

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>list</title>
    <!-- SEO优化 -->
    <meta name="description" content="" />
    <meta name="keywords" content="" />
    <!-- 默认使用360极速模式加载 -->
    <meta name="renderer" content="webkit">
    <meta http-equiv="X-UA-Compatible" content="IE=Edge,chrome=1">
    <!-- 样式引入 -->
    <link rel="stylesheet" type="text/css" href="css/common/base.min.css">
    <link rel="stylesheet" type="text/css" href="css/list/list.less">
    <link rel="stylesheet" type="text/css" href="css/header/heater.less">
    <link rel="stylesheet" type="text/css" href="css/header/banner.less">
     <link rel="stylesheet" type="text/css" href="css/foot/footer.less">
     <link rel="stylesheet" type="text/css" href="css/page//page.less">

</head>
<body>
    <link rel="import" href="widget/header/header.html?__inline">
    <link rel="import" href="widget/header/banner.html?__inline">

    <div class="listMain">
         <link rel="import" href="widget/list/listLiftImg.html?__inline">
         <link rel="import" href="widget/list/listRight.html?__inline">
         <div class="c"></div>
          <link rel="import" href="widget/page/page.html?__inline">
    </div>
     <link rel="import" href="widget/foot/footer.html?__inline">
</body>
</html>
```

>[查看更多源码](https://github.com/cmh2016/code/tree/master/fis3)

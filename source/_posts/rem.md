---
title: rem 实现手机屏幕适配
date: 2016-03-15 16:53:40
tags: [css3,rem,webapp]
categories: webapp
---
常用单位
======
> 先看一下px，em，rem，%之间的关系

* px：就是像素，相对于屏幕分辨率而言的，简单来说就是平时我们写静态页面时用px作单位

* 百分比（%）：这比较适合块状的页面设计，比如经常在写移动端的时候也会用到，经常做响应式设计。例如经典的bootstrap的栅格系统就是通过css3的媒体查询定义不同的`col-md-*`的宽度百分比值。感兴趣的可以看一下bootstrap源码
* em：是相对长度单位，相对于父级元素的单位。如当前对行内文本的字体尺寸未被人为设置，则相对于浏览器的默认字体尺寸。就是当默认字体大小为14px，那么就有1em=14px；

* rem：是相对于根元素，这样就意味着，我们只需要在根元素确定一个参考值。

rem优势和使用场景
=======
* 最经典的实现响应式方案，就拿bootstrap来实现移动端的响应式来说，我们一般根据媒体查询做iPhone5s，iPhone6s，iPhone6 plus等几套css样式。

* 因为rem是根据html的`font-size`来自动计算。所以，我们只需要使用媒体查询定义常用的屏幕规定html根字体就可以。
> 例如

```
html {
    font-size: 20px;
    -ms-text-size-adjust: 100%;
    -webkit-text-size-adjust: 100%
}

@media only screen and (min-width: 400px) {
    html {
        font-size:21.33333333px!important
    }
}

@media only screen and (min-width: 414px) {
    html {
        font-size:22.08px!important
    }
}

@media only screen and (min-width: 480px) {
    html {
        font-size:25.6px!important
    }
}
```
比如，1rem默认为20px；
作为尺寸单位，可以作为字体大小，亦可作为盒子模型的属性。
效果演示
======
![](http://ol1kqeyve.bkt.clouddn.com/17-2-8/4738033-file_1486545623680_17a93.gif)

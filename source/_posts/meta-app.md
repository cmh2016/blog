---
title: 移动端的奇淫技巧
date: 2016-04-08 17:21:28
tags: [webapp,css3,响应式]
categories: webapp
---
viewport和meta
===========
```
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <!--针对ios-->
    <meta content="yes" name="apple-mobile-web-app-capable">
    <!--添加主屏后再次打开是否全屏-->
    <meta name="apple-touch-fullscreen" content="yes">
    <!--禁止百度转码-->
    <meta name="Cache-Control" content="no-siteapp">
    <!--ios顶部颜色-->
    <meta content="black" name="apple-mobile-web-app-status-bar-style">
    <!--禁用数字识别电话-->
    <meta content="telephone=no" name="format-detection">
    <!--添加主屏后显示标题文字-->
    <meta name="apple-mobile-web-app-title" content="卡券商城">
    <!--添加主屏显示的图标 sizes为尺寸-->
    <link rel="apple-touch-icon" sizes="144x144" href="">
    <!--viewport-->
    <meta content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0" name="viewport">

```
CSS伪类:active
============
如果你想使用元素的伪类来实现 按下激活 状态，那么你需要知道以下问题：

* iOS上的几乎任何浏览器，定义元素的伪类 :active 都是无效；
Android上，Android Browser 和 Chrome 都支持伪类 :active ，其它第三方浏览器有部分不支持；

* 定义了 :active 并且当前浏览器环境支持，当手指在滚动或者无意间的划过时，:active 状态都会被激活；

> 为了规避上述所有的问题，如果需要 按下激活 状态，推荐使用 js 新增一个 className

清除输入框内阴影
===========
iOS上的几乎任何浏览器输入框（input, textarea）默认有内部阴影，但无法使用 box-shadow 来清除，如果不需要阴影，可以这样关闭：
```
input,
textarea {
    /* 方法1: 去掉边框 */
    border: 0;

    /* 方法2: 边框色透明 */
    border-color: transparent;

    /* 方法3: 重置输入框默认外观 */
    -webkit-appearance: none;
    appearance: none;
}
```
禁止保存或拷贝图像
===========
```
img {
    -webkit-touch-callout: none;
}
```

取消touch高亮
==========
在移动设备上，所有设置了伪类 :active 的元素，默认都会在激活状态时，显示高亮框，如果不想要这个高亮，那么你可以通过以
> 使用下方法来禁止：

```
.xxx {
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
}
```

禁止选中内容
==========
如果你不想用户可以选中页面中的内容，那么你可以禁掉：
```
html {
    -webkit-user-select: none;
}
```

快速回弹滚动
=========
早期的时候，移动端的浏览器都不支持非body元素的滚动条，所以一般都借助 iScroll;
Android 3.0/iOS解决了非body元素的滚动问题，但滚动条不可见，同时iOS上只能通过2个手指进行滚动；
Android 4.0解决了滚动条不可见及增加了快速回弹滚动效果，不过随后这个特性又被移除；
iOS从5.0开始解决了滚动条不可见及增加了快速回弹滚动效果
在iOS上如果你想让一个元素拥有像 Native 的滚动效果
> 你可以这样做：

```
.xxx {
    overflow: auto; /* auto | scroll */
    -webkit-overflow-scrolling: touch;
}
```
开启拨打电话功能：
========
`<a href="tel:13256485323">拨打</a>`
开启发送短信功能：
========
`<a href="sms:13256485323">发送短信</a>`

开启邮件发送：
=========
`<a href="mailto:dooyoe@gmail.com">dooyoe@gmail.com</a>`

禁止文本缩放
=========
当移动设备横竖屏切换时，文本的大小会重新计算，进行相应的缩放，当我们不需要这种情况时，可以选择禁止：
```
html {
    -webkit-text-size-adjust: 100%;
}
```

webkit表单元素重置
=====================
`.css{-webkit-appearance:none;}`
webkit表单输入框placeholder
====================
```
input::-webkit-input-placeholder{color:#AAAAAA;}
input:focus::-webkit-input-placeholder{color:#EEEEEE;}
```

开启硬件加速
=========
> 解决页面闪白 保证动画流畅

```
.xxx {
   -webkit-transform: translate3d(0, 0, 0);
   -moz-transform: translate3d(0, 0, 0);
   -ms-transform: translate3d(0, 0, 0);
   transform: translate3d(0, 0, 0);
}
```

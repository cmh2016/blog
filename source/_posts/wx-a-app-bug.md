---
title: 微信小程序--那些年踩过的坑
date: 2017-02-8 16:04:57
tags: [微信小程序,webapp]
categories: 微信小程序
---
这就尴尬了
============
> 很多人理直气壮的说，微信小程序不就是html+css+js
吗？好吧，我原谅你的无知。如果真的这样认为就要认真看一下，不要踩坑

* 微信小程序里的js不是js。程序屏蔽了一部分原生的 API 接口，以下为被屏蔽：window document frames self location navigator localStorage history Caches screen alert confirm prompt XMLHttpRequest WebSocket。所以不要再问为什么我alert不能用。呵呵

* wxss不支持less/sass等css预处理语言

* 网络请求报非法是，修改header的content-type属性如下（[其他说法](https://www.zhihu.com/question/51145570))
>注意，前提是无appId。如果有，报错不在合法请参考官网
    ```
    header: {
      'content-type': 'application/x-www-form-urlencoded'
    }
    ```
* 不支持高级的css原则器。0.0(官方说的是目前)
    >仅支持以下xss选择器
    .class #id element element,element ::after ::before
    所以不要使用> + ~ nth-child ....

* 这个不算是坑，因为微信小程序是单向数据绑定，所以在mvvm模式下，记得修改数据时及时setData

* 还有就是官方给的IDE，经常出现莫名其妙的问题。比如，你修改保存后，页面没有修改亦或卡死等情况

* 新建页面的时候，一定要按照4个文件的要求，然后切记在app.js的page项中注册，否则报错

* 导航栏文字颜色只支持黑/白，但是测试中发现本地下可以任意颜色，但是发布后，颜色就会变为默认的白色，这点注意
    >有图有真相！
    ![](http://ol1kqeyve.bkt.clouddn.com/17-2-10/19089130-file_1486719609301_55e5.png)

---
title: 微信公众号开发之LocalStorage
date: 2018-06-21 15:53:11
tags: [wechat,webapp]
categories: webapp
comments: true
---

>最近开发一个公众号，使用LocalStorage记录是否是第一次打开和用户的登录状态，用微信的OAuth2.0鉴权实现多终端多设备共享登录状态。

***

##### 思路是LocalStorage里面储存用户登录标记及相关信息，有的话进行业务，没有的话从数据库查找

###### 具体思路为：思路是LocalStorage里面储存用户登录标记及相关信息，有的话进行业务，没有的话从数据库查找    
- 首先使用静默授权[【不清楚看这里】](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140842)获取openid，使用功能时先check下LocalStorage里有没有登录状态，没有的话跳转登录绑定openid和用户信息储存数据库，并将用户状态和登录状态储存在localstore里面。
- 然后，每次进入redirect_uri的时候如果本地缓存有不是第一次登录的话直接显示登录及用户信息，如果没有的话从数据库查询并放入本地缓存。
- 最后，退出登录的时候clear掉LocalStorage，然后再次进入时走上一步，行程一个闭环。
- **使用第一次登录的LocalStorage标记主要是为了减少接口的调用，不用每次都从数据库拿信息。减少服务器开销**

##### 开发过程中偶尔遇到LocalStorage被清除的，难道微信浏览器对LocalStorage有“特殊对待”？于是进行的百度，得到了下面的结果：

> *微信内置浏览器按照微信团队的说法就是标准的 WebView（Android下），跟普通 webAPP 本质上是一样的。当 WebView 因为内存不足、进程被杀、微信主动杀掉等原因被干掉以后，所有跟浏览器相关的信息全部灰飞烟灭，cookie、LocalStorage、SessionStorage、WebSQL 全部消失。*

![](http://ww4.sinaimg.cn/large/9150e4e5ly1fmed0xxbcyg206o06oglr.gif)

经过一上午的测试、修改、排查，发现ios下通用里面的清除缓存也无法清理掉LocalStorage！神马情况，网上都是莫名被清除，为啥我的是相反的结果？！然后怎么搞都没清除掉LocalStorage，就连杀微信进程也没清除LocalStorage？最后神奇的发现微信账号退出后，LocalStorage里面没值了！！！

![](http://ww2.sinaimg.cn/large/6af89bc8gw1f8no4n150gj204g04xjrb.jpg)

经过细心的翻阅官方指摘发现这么一句话[“退出微信账号后，将会清空所有Cookie和LocalStorage”](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1483682025_enmey)，不清楚网上说的是否在安卓或其他设备上出现过，**仅测试iPhone6**。

写的比较乱，大家见谅。仅此为以后入坑的开发者作为参考。
![](http://ww2.sinaimg.cn/large/6af89bc8gw1f8o4hpr55vj208e07xq32.jpg)

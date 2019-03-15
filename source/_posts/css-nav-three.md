---
title: 纯css实现3级导航菜单
date: 2016-03-10 14:07:32
tags: [css3,nav]
categories: css3
---
优势
=====
* 抛弃繁冗的jqurey和第三方插件，加快dom渲染速度

* 书写简单，掌握html和css即可

* 使用css3过渡阴影等属性，更加流畅

* 结构清晰，模块化
实现思路
======
1. 使用html元素的继承性和hover伪类

2. 使用`position`定位元素

3. 结构如下
    `<li>
    <a href="">机构设置</a>
    <!--二级导航-->
    <ul class="navTowList">
        <li>
            <a href="">行政管理<span class="more">&gt;</span></a>
            <!--三级导航-->
            <ul class="navThreeList">
                <li><a href="">行政办公室</a></li>
                <li><a href="">学生工作办公室</a></li>
            </ul>
        </li>
        <li>
            <a href="">系、部、中心<span class="more">&gt;</span></a>
            <!--三级导航-->
            <ul class="navThreeList">
                <li><a href="">自动化系</a></li>
                <li><a href="">通信工程系</a></li>
                <li><a href="">计算机系</a></li>
                <li><a href="">计算机基础教学部</a></li>
                <li><a href="">计算机实验教学中心</a></li>
                <li><a href="">通信工程实验室</a></li>
                <li><a href="">自动化实验室</a></li>
            </ul>
        </li>
        <li>
            <a href="">培训机构<span class="more">&gt;</span></a>
            <!--三级导航-->
            <ul class="navThreeList">
                <li><a href="">计算机等级考试</a></li>
                <li><a href="">工程硕士入学考试</a></li>
                <li><a href="">卫生专业技术资格考试</a></li>
                <li><a href="">计算机软件水平考试</a></li>
                <li><a href="">全国中小学教师计算机水平考试</a></li>
            </ul>
        </li>
    </ul>
</li>`
效果
=====
![](http://ol1kqeyve.bkt.clouddn.com/17-2-8/2507543-file_1486535878937_6b41.gif)
获取源码
=======
[github](https://github.com/cmh2016/code/tree/master/reset-web)

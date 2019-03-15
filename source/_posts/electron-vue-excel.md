---
title: electron-vue-excel
date: 2018-08-21 09:03:25
tags: [vue,electron,node]
---

### elecrton 是什么？
[摘自官网]
*Electron是由Github开发，用HTML，CSS和JavaScript来构建跨平台桌面应用程序的一个开源库。 Electron通过将Chromium和Node.js合并到同一个运行时环境中，并将其打包为Mac，Windows和Linux系统下的应用来实现这一目的。*

*简而言之：使用 JavaScript, HTML 和 CSS 构建**跨平台**的桌面应用*
[electron中文官网地址](https://electronjs.org/)

#### elecrton 有什么特点，前端选择electron开发桌面应用有什么优势？

* web 技术
>Electron 基于 Chromium 和 Node.js, 让你可以使用 HTML, CSS 和 JavaScript 构建应用。所以对于前端工程师来说有着近水楼台的先天优势。
* 开源，跨平台
>如果您想在linux、mac、window下开发统一UI及功能的桌面应用，使用electron可以方便快捷的build成不同平台的安装包。
* nodeJs的完美融合
>引用npm官网的一句话“Build amazing things”，在项目中可以使用前端的任何框架和npm里面所有的模块，比如：ffi模块调取dll文件使你的应用原生功能更加强大，扩展性更好。

#### electron与nw.jsd该怎么选择？
项目不需要兼容WinXP ? 果断选择electron : nw.js
 ***

###使用electron-vue 快速搭建项目
*[摘自官网]electron-vue 充分利用 vue-cli 作为脚手架工具，加上拥有 vue-loader 的 webpack、electron-packager 或是 electron-builder，以及一些最常用的插件，如vue-router、vuex 等等。*

####项目从开发到打包安装文件
1.1 项目使用electron-vue快速搭建，开发中使用yarn管理依赖[如果游vpn的话随意]

1.2 项目UI使用vue的UI库，element、iview根据喜好

1.3 使用node-xlsx读取和生成excel文件

2.1 项目目的：对比两个excel文件中某一列相同项，并且支持追加导出功能。

2.2 开发遇到的问题：node-xlsx读取excel为二维数组，使用element table组件需要数组对象的json格式，所以代码里很多for循环，为了优化大文件的效率，建议巧妙使用if判断和for循环优化写法。

2.3 打包遇到的问题：本机开发打包白屏，但是run dev正常。请参考webpack.renderer.config.js第110行 。 第二种：本机打包正常，copy给其他电脑白屏，报各种模块不存在，请使用【npm】打包！！！一切问题迎刃而解。怀疑是yarn安装依赖没有被打包。

3.1 使用NSIS打包electron生成的可执行文件，需要注意的是在 5/8 的时候先移出全部在依次点击第一个添加exe文件，再点击第二个选择打包后exe所在的目录{记得勾选包含子目录}，然后编译，就会生成一个exe安装文件。

####项目截图
![](http://ol1kqeyve.bkt.clouddn.com/%E6%8D%95%E8%8E%B7.PNG)
![](http://ol1kqeyve.bkt.clouddn.com/TIM%E6%88%AA%E5%9B%BE20180821092855.png)
![](http://ol1kqeyve.bkt.clouddn.com/TIM%E6%88%AA%E5%9B%BE20180821093600.png)
![](http://ol1kqeyve.bkt.clouddn.com/TIM%E6%88%AA%E5%9B%BE20180821093641.png)
![](http://ol1kqeyve.bkt.clouddn.com/TIM%E6%88%AA%E5%9B%BE20180821093703.png)

####项目地址
[安装包下载](http://ol1kqeyve.bkt.clouddn.com/excel-tools.exe)
[github](https://github.com/cmh2016/electron-excel-tool)
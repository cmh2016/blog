---
title: javascript+ajax+json+vue实现银行卡号归属地查询
date: 2016-03-08 11:13:45
tags: [javascript,json,ajax,vue.js]
categories: javascript
---
使用场景
=========
*  根据银行卡号返回正确的发卡行名称和银行卡类型
技术栈
===========
* json数据使用
* ajax获取json
* 使用vue.js的mvvm模式，简单的双向数据绑定
核心代码
========
>对比用户输入和json数据对比进行判断,注意本地使用注意跨域
```
function bankInfo() {
  var inputValue = $("#account").val();
  var InputValue = inputValue.replace(/\s/g, "");
  console.log(InputValue);
  var tipiofo = $(".tipiofo");
  if (InputValue == "") {
    alert("银行卡号不能为空！");
    $("#account").focus();
  } else {
    $.getJSON("data/bankData.json", function(data) {
      for (var num in data) { //遍历json数组
        if (InputValue.substring(0, 4) == data[num].bin || InputValue.substring(0, 6) == data[num].bin || InputValue.substring(0, 7) == data[num].bin || InputValue.substring(0, 8) == data[num].bin || InputValue.substring(0, 5) == data[num].bin || InputValue.substring(0, 9) == data[num].bin) {
          tipiofo.text(data[num].bankName);
        }
      }
    });
  }
  if (tipiofo.text() == "") {
    tipiofo.text("其他银行");
  }
}
```
效果演示
=======
![](http://i1.piimg.com/567571/6041b92e40c6298f.gif)
获取代码
========
[github](https://github.com/cmh2016/code/tree/master/bank)

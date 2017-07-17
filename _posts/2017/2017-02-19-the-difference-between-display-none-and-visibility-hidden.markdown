---
layout:     post
title:      "display: none和visibility: hidden的区别"
subtitle:   "what's the difference between display: none and visibility: hidden?"
date:       2017-02-19
author:     "Eric"
header-img: "img/2017-plan.jpg"
catalog: true
tags:
    - 区别
    - 隐藏
---


display: none和visibility: hidden两个CSS样式都能实现隐藏的目的，但它们是不同的，本文将解释一下他们之间的区别。

**visibility: hidden**隐藏了元素，但它仍旧在文档中占着位置。换句话说，假如你写了一个DIV，给它一个100×100px大小，visibility: hidden属性将会使DIV隐藏，但它后面的文字将不会变动其位置。

**display: none**会把元素移除文档。它不会占任何位置，即使元素的HTML标签还在源代码里。在上面例子中，随着DIV被隐藏，DIV后面的文字将移动上来。



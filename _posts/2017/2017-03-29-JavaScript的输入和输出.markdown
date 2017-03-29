---
layout:     post
title:      "JavaScript的输入和输出"
subtitle:   " "
date:       2017-03-29
author:     "Eric"
header-img: "img/2017-plan.jpg"
catalog: true
tags:
    - JavaScript
    - 输入
    - 输出
---

# JavaScript的输入和输出
JavaScript有很多种输入和输出方式，具体应用需要看项目的需求和具体的用途，比如测试代码的输出结果是否正确，可以用`alert()`弹出代码的运算结果。本文就这些输入和输出方式做一下总结，方便大家用到的时候参考。
## 输入
js的输入主要有`prompt()`，该方法提供了最简单的信息输入方式，其基本格式如下：Window.prompt("提示信", 预定输入信息);此方法首先在浏览器窗口中弹出一个对话框, 让用户自行输入信息。一旦输入完成后，就返回用户所输入信息的值。例:
`test=prompt（“请输入数据:”，”this is a JavaScript”）`。

由于其对象是window，所以window可以省略。该方法会返回用户填入的信息，一般可以用一个变量储存它，但需注意其返回的是一个字符串。
## 输出
   js的输出方式主要有：`alert()`、`document.write()`、`console.log()`。`alert()`是window的一个方法，所以也可以省略window，它的主要作用是在输出时产生一个警告窗口，提示信息或提醒用户。同时在程序的调试过程中，也可以用它来查看一段程序的输出结果是否正确，但一般比较少采用这种方式。
   
   `document.write()`是document的一个方法，前面document不能省略，主要是向浏览器窗口中输出文本字符串，其还有另外一种写法是`document.writeln()`，这种写法会自动加个回车符。该方法也可以用于测试程序。
   
   `console.log()`是向控制台（console）输出结果，该方法是在调试程序中用到最多的一种方法。
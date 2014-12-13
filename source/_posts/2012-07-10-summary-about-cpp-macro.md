---
layout: post
title: "Macro 小总结"
date: 2012-07-10
comments: true
tags: CPP
---
如果在C++中使用Macro，要注意：<br />1. parameter要加括号： #define ADD(x,y) ((x)+(y))<br />2. result要加括号： #define ADD(x,y) ((x)+(y))<br />3. 多行要加花括号： #define INCREASE(a, b) {++(a);++(b);}<br /><br />这里有个例子：https://github.com/fresky/MacroExample<br />
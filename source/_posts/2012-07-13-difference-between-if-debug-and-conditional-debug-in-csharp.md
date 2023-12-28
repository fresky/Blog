---
layout: post
title: "C#中 #if DEBUG 和 Conditional(\"DEBUG\")的区别"
date: 2012-07-13
comments: true
tags: Programming
---
这里<a href="http://stackoverflow.com/questions/3788605/if-debug-vs-conditionaldebug">c# - #if DEBUG vs. Conditional("DEBUG") - Stack Overflow</a>解释了两者的区别。摘要如下：<br />#if DEBUG: 发生在编译时，release编译出的IL不包含if中的代码<br />[Conditional("DEBUG")]: 发生在运行时，releae编译出的IL包含代码，但是不会被执行。<br /><blockquote></blockquote>
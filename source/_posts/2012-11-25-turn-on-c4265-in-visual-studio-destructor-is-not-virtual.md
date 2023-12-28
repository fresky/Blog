---
layout: post
title: "Visual Studio中打开编译器中有虚函数的类的析构函数不是虚函数的警告——/we4265"
date: 2012-11-25
comments: true
tags: Programming
---
C++中一个类如果有虚函数，那么它的析构函数应该也是虚的，否则会出现很多问题。<br />其实visual studi中有一个专门的warning是给这个的，<a href="http://msdn.microsoft.com/en-us/library/wzxffy8c%28v=vs.110%29.aspx">Compiler Warning (level 3) C4265</a>：'class' : class has virtual functions, but destructor is not virtual<br />但是这个warning默认是关闭的，这里可以看所有默认关系的warning：<a href="http://msdn.microsoft.com/en-us/library/23k5d385(v=vs.110).aspx">Compiler Warnings That Are Off by Default</a><br />我们可以指定编译器选项 /we4265 把特定的warning打开，具体见<a href="http://msdn.microsoft.com/en-us/library/thxezb7y(v=vs.110).aspx">/w, /Wn, /WX, /Wall, /wln, /wdn, /wen, /won (Warning Level)</a>。<br /><blockquote></blockquote><br /><blockquote></blockquote>
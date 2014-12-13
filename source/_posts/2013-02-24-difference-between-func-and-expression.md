---
layout: post
title: "C#中Func和Expression的区别"
date: 2013-02-24
comments: true
tags: CSharp
---
<p>LINQ中IEnumerable&lt;T&gt;的where接受的是Func，但是IQueryable&lt;T&gt;接受的是Expression。</p>  <p>区别在于Func直接会被编译器编译成IL代码，但是Expression只是存储了一个表达式树，在运行期作处理。比如在LINQ TO SQL的时候就可以把这个表达式树变成sql语句。</p>  <p>可以调用Expression的Compile方法，把一个Expression编译成一个Func，反之则没有，不能把一个Func转换成Expression。</p>
---
layout: post
title: "C#的内存模型和并发情况下受到的影响"
date: 2012-12-26
comments: true
tags: CSharp
---
<a href="http://msdn.microsoft.com/en-us/magazine/jj863136.aspx">C# - The C# Memory Model in Theory and Practice</a>讲了C#的内存模型和在并发下的影响。<br />内存操作重排：当一个线程读一段内存，如果这段内存同时被另外一个线程写，那么读的线程有可能拿到一个不新鲜的值。<br />用volatile关键字可以限制内存重拍。<br />原子读写操作：reference，bool，char，byte，sbyte，short，ushort，unit，int，float。<br />非重排优化，例如读一个field，然后存在一个variable中，后面被多次读到，有可能被优化成没有variable，直接多次从field中读取。<br /><br />线程交换的模式：<br />锁<br />通过Threading API<br />通过type初始化（static field）<br />通过volatile的filed<br />lazy initialize<br />Interlocked <br />使用Concurrency Primitives<br /><br /><br /><br /><br /><br /><blockquote></blockquote>
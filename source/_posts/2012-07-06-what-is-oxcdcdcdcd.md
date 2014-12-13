---
layout: post
title: "oxcdcdcdcd是什么？"
date: 2012-07-06
comments: true
tags: Debug
---
今天看到一个crash的dump，crash发生在尝试释放地址，但是从dump上可以看到地址的值是oxcdcdcdcd。从这可以知道这个地址没有被正确的初始化。还是debug编译出来的好啊：）<br />从网上总结了一下地址的信息，最主要的是：<br /><table><tbody><tr><td>0xCDCDCDCD</td><td>堆上分配的地址，但是没有初始化<br /></td></tr>

<tr><td>0xDDDDDDDD</td><td>堆上释放的地址。</td></tr>

<tr><td>0xFDFDFDFD</td><td>堆内存的边界<br /></td></tr>

<tr><td>0xCCCCCCCC</td><td>栈上分配的内存，但是没有初始化</td></tr></tbody></table><br />更多的内容可以看看下面2篇文章。<br /><a href="http://www.nobugs.org/developer/win32/debug_crt_heap.html">Win32 Debug CRT Heap Internals</a><br /><blockquote></blockquote><a href="http://www.highprogrammer.com/alan/windev/visualstudio.html">Microsoft Visual C++ Tips and Tricks</a><br /><blockquote></blockquote>
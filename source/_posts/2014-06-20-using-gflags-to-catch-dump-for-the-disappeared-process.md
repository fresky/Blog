---
layout: post
title: "消失的进程——谁是凶手?"
date: 2014-06-20 16:10
comments: true
tags: Debug
keywords: gflags, windbg, dump, process,
description: 使用gflags的Monitoring Silent Process Exit在进程退出前抓取dump，分析进程神秘消失的原因。
---
最近遇到一个奇怪的问题，在用TestComplete进行自动化测试时，一个进程莫名其妙的退出了，没有留下任何痕迹。首先想到的是用Windbg attach到这个进程尝试看看是什么原因导致进程退出。但是一旦用Windbg attach，就不能重现了。于是就需要祭出本文的主角：[gflags](http://msdn.microsoft.com/en-us/library/windows/hardware/ff549557%28v=vs.85%29.aspx)。gflags可以开启或者关闭很多调试开关，本文就介绍一下gflags的[Monitoring Silent Process Exit](http://msdn.microsoft.com/en-us/library/windows/hardware/jj602791%28v=vs.85%29.aspx)的功能。

从Windows7开始，可以使用gflags的Monitoring Silent Process Exit设置来监视进程因为如下两个原因退出。  
  - 自己终结。进程自己调用了`ExitProcess`。可以通过设置Ignore Self Exits来忽略这种情况的退出。  
  - 跨进程终结。别的进程调用了`TerminateProcess`。

使用方法很简单，打开gflags，点击Monitoring Silent Process Exit页面，输入想监视的进程，然后敲击tab，把Enable Monitoring Silent Process Exit Monitoring打勾就行了，如下图所示。

{% limg gflags.png %}

gflags可以设置3种报告模式：  
  - Launch monitor process：启动一个监视进程，例如可以在进程退出时启动windbg。用`windbg -p %e`来打开windbg，并且attach到目标进程上。    
  - Enable dump collection：自动收集dump。  
  - Enable notification：桌面通知。   

配置好这些之后，在进程退出时就能收到通知，并且调试了。也会在Event Viewer里记录一条记录，也可以通过打开`eventvwr.msc`，在Windows Logs > Application中找Source是Process Exit Monitor的。

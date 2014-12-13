---
layout: post
title: ".NET 2.0的Timer elapsed event 会自动catch住所有的exception"
date: 2011-06-23
comments: true
tags: CSharp
---
今天调试一个Access Violation的问题，用VS attach到程序上，打开所有的exception，结果无意中抓到了一个NullRefereceException，从一个Timer.Elapsed的event handler中抛了出来，很奇怪我没有try catch，但是怎么没有crash呢，查了一下MSDN<a href="http://msdn.microsoft.com/en-us/library/system.timers.timer%28v=vs.85%29.aspx">Timer Class (System.Timers)</a>发现了下面这句话，原来在.NET 2.0之前会自动的catch住所有的exception，大家以后写程序注意一下。<br /><br />In the .NET Framework version 2.0 and earlier, the <b>Timer</b> component catches and suppresses all exceptions thrown by event handlers for the <b>Elapsed</b> event. This behavior is subject to change in future releases of the .NET Framework.<br />

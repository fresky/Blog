---
layout: post
title: "用LINQPad加上Tx驱动来分析log"
date: 2014-01-09 12:09
comments: true
tags: [Programming, Tool]
---

[Tx (LINQ to Logs and Traces)](http://tx.codeplex.com/)是[微软发布的开源工具](http://blogs.msdn.com/b/interoperability/archive/2014/01/06/new-release-tx-linq-to-logs-and-traces.aspx)。可以用这个工具来使用LINQ分析日志，包括

1. Event Tracing for Windows (ETW)
2. Event Logs (.etvtx)
3. Performance counters from files (.blg, .csv, .tsv)
4. IIS text logs in W3C format
5. SQL Server Extended Events (XEvent)

可以自己写代码通过NuGet加上Tx相关的reference，或者更方便的使用[LINQPad](http://www.linqpad.net/)。在LINQPad中加上Tx的驱动，就可以使用了，具体参见[LINQPad driver for Tx](http://tx.codeplex.com/wikipage?title=LINQPad%20Driver&referringTitle=Documentation)，还可以下载一个Tx的Sample来学习。
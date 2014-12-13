---
layout: post
title: "记录C#程序的运行时间"
date: 2012-07-06
comments: true
tags: CSharp
---
从<a href="http://stackoverflow.com/questions/11318175/does-a-c-sharp-app-track-how-long-its-been-running?newsletter=1&amp;nlcode=55866%7cc739">.net - Does a C# app track how long its been running? - Stack Overflow</a>看到的，很方便的。<br /><code>System.Diagnostics.Process</code> 有个属性记录的应用的开始时间。<br />

```c#
System.Diagnostics.Process current = System.Diagnostics.Process.GetCurrentProcess();
DateTime startedTime = current.StartTime;
```
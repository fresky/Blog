---
layout: post
title: "c# lock 使用不当引起的死锁"
date: 2012-07-19
comments: true
tags: CSharp
---
[Thread Synchronization (C# and Visual Basic)](http://msdn.microsoft.com/en-us/library/ms173179%28v=vs.110%29.aspx)中提到最好不要lock public的东西，比如：

1. `lock（this）`<br />2. `lock（“string”）`<br />3. `lock（typeof（int))`<br />我在<a href="https://github.com/fresky/DeadLock">github</a>上放了个死锁的例子。<br />
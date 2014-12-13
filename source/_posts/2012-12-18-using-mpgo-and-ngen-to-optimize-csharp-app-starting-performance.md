---
layout: post
title: "使用MPGO和NGEN来优化C#桌面程序的启动性能"
date: 2012-12-18
comments: true
tags: CSharp
---
<p>C#桌面程序可以通过NGen创建本机映像（包含经编译的特定于处理器的机器代码的文件），并将它们安装到本地计算机，这样在运行时可从缓存中使用本机映像，而不必使用实时 (JIT) 编译器编译原来的IL代码。具体参见<a href="http://msdn.microsoft.com/en-us/magazine/cc163610.aspx">CLR Inside Out: The Performance Benefits of NGen.</a></p>  <p>在Visual Studio2012中，有一个新的工具可以进一步优化启动性能，叫做<a href="http://msdn.microsoft.com/en-us/library/hh873180.aspx">Mpgo.exe (Managed Profile Guided Optimization Tool)</a>。它的工作原理就是先用MPGO跑一遍应用程序，然后把收集到的性能数据放到IL中，这样NGEN再生成本机映像时可以利用这些信息做进一步的优化。具体参见：<a href="http://blogs.msdn.com/b/dotnet/archive/2012/03/20/improving-launch-performance-for-your-desktop-applications.aspx">Improving Launch Performance for Your Desktop Applications</a>。</p>
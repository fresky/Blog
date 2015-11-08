---
layout: post
title: "C#的一个小函数来计算一个运算使用的时间和内存"
date: 2013-06-08
comments: true
tags: CSharp
---
<p>&#160;</p>  <p>如下，分别计算时间和内存。</p>  

```csharp
        public static double TimeWatcher(Action action)
        {
            System.Diagnostics.Stopwatch watch = new System.Diagnostics.Stopwatch();
            watch.Start();
            action();
            watch.Stop();
            var useTime = (double) watch.ElapsedMilliseconds/1000;
            return useTime;
        }

        public static long MemoryWatcher(Action action)
        {
            long start = GC.GetTotalMemory(true);
            action();
            GC.Collect();
            GC.WaitForFullGCComplete();
            long end = GC.GetTotalMemory(true);
            long useMemory = (end - start)/(1024*1024);
            return useMemory;
        }
```
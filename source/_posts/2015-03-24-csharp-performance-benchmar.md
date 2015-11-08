title: 针对C#程序做性能测试的一些基本准则
date: 2015-03-24 23:20:20
categories:
tags: [CSharp, Testing]
description: 
---

Eric Lippert在他的[C# Performance Benchmark Mistakes, part1](http://tech.pro/blog/1293/c-performance-benchmark-mistakes-part-one)，[part2](http://tech.pro/tutorial/1295/c-performance-benchmark-mistakes-part-two)，[part3](http://tech.pro/tutorial/1317/c-performance-benchmark-mistakes-part-three)，[part4](http://tech.pro/tutorial/1433/performance-benchmark-mistakes-part-four)中介绍了一些最基本的针对C#的程序做性能测试的准则，要点如下：

1. 选择正确的衡量标准。  
2. 在做子系统局部性能测试的同时不要忘记花点时间做整个系统的端到端的性能测试。  
3. 不要在debugger中做性能测试。  
4. 不要在debug编译模式下做性能测试。  
5. 使用Stopwatch，不要使用DateTime。  
6. 在性能测试时要对第一次运行区别对待，因为第一次运行会牵扯到JIT即时编译。如果要测试启动性能，那么只需要测试第一次运行时。  
7. 在和软件最终运行环境一样的软硬件条件下做性能测试。这点和第3、4点其实说的是一个道理，测试环境要和软件真正的工作环境一致。  
8. 考虑垃圾回收带来的影响，一个可能的做法是在每次测试之前和之后用下面的代码强制调用一下GC。  
```csharp
System.GC.Collect();
System.GC.WaitForPendingFinalizers();
System.GC.Collect();
```
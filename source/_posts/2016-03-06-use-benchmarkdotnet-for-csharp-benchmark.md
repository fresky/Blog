title: 用BenchmarkDotNet给C#程序做性能测试
date: 2016-03-06 10:22:53
categories:
tags: [Tool, CSharp]
description:
---

[BenchmarkDotNet](https://github.com/PerfDotNet/BenchmarkDotNet)是一个用MIT协议开源的C#程序性能测试的一个库，非常简单易用。

# 用法
1. 安装NuGet包，BenchmarkDotNet  
1. 在需要做性能测试的方法前加上属性`[Benchmark]`。  
1. 在Main函数调用性能测试`var summary = BenchmarkRunner.Run<Md5VsSha256>();`。  

# 工作原理
1. BenchmarkDotNet为每一个要做性能测试的方法生成了一个单独的项目，放在程序的输出目录下，用Release模式编译，并且运行多次来统计性能测试结果。    
1. 每次运行都包含以下几个步骤。  
    * Pilot： 决定运行几次。  
    * IdleWarmup， IdleTarget：评估BenchmarkDotNet这个工具带来的额外开销。  
    * MainWarmup：测试热身。  
    * MainTarget：测试。  
    * Result：测试结果减去BenchmarkDotNet带来的额外开销。  
1. 生成测试报告。有各种格式，包括html格式，markdown格式（缺省风格，github风格，stackoverflow风格），txt格式，csv格式。比如如下就是我运行示例代码后得到的github风格的输出。

```ini
BenchmarkDotNet=v0.9.2.0
OS=Microsoft Windows NT 6.1.7601 Service Pack 1
Processor=Intel(R) Core(TM) i7-4600U CPU @ 2.10GHz, ProcessorCount=4
Frequency=2630693 ticks, Resolution=380.1280 ns
HostCLR=MS.NET 4.0.30319.42000, Arch=32-bit RELEASE

Type=Md5VsSha256  Mode=Throughput  

```
 Method |      Median |    StdDev |
------- |------------ |---------- |
    Md5 |  21.8816 us | 0.6091 us |
 Sha256 | 123.4171 us | 6.7846 us |

可以在程序的输出目录下的log文件中看到上面所说的每个过程的详细信息。

# 其他配置
BenchmarkDotNet还有很多可以配置的地方，可以参见主页的介绍。
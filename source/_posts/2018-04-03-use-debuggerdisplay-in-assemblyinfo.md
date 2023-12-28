title: 用DebuggerDisplay在Visual Studio的调试器中定制类的显示方式
date: 2018-04-03 20:58:36
categories:
tags: [Debug, Programming]
description:
---
我在之前的博客[用Natvis定制C++对象在Visual Studio调试时如何显示](/2015/06/17/customize-your-debugger-for-cpp-object/)和[定制自己的Visual Studio的Debugger Visualizer](/2012/07/16/customize-your-own-debugger-visualizer-in-csharp/)中介绍过如何用[Debugger Display Attributes](https://docs.microsoft.com/en-us/visualstudio/debugger/using-the-debuggerdisplay-attribute)来定制C++和C#的对象在Visual Studio调试时如何显示。这个`DebuggerDisplay`也可以直接放在C#的`AssemblyInfo.cs`中来指定当调试当前程序时在Watch窗口如何显示其它的类型。

比如在我们调试WMI时，想找一个属性，看到如下的调试窗口，然后一个个的手动展开，估计是要崩溃的。

{% limg DebuggerDisplay1.png %}

但是如果我们在`AssemblyInfo.cs`中写了如下的代码，那么看起来就顺眼多了。

```csharp
[assembly: DebuggerDisplay("[Name={Name}, Value={Value}]", Target = typeof(System.Management.PropertyData))]
```

{% limg DebuggerDisplay2.png %}
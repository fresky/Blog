---
layout: post
title: "C#的强迫执行域Constrained Execution Regions(CERs)"
date: 2013-06-07
comments: true
tags: [CSharp, Debug]
---
<p>强迫执行域（CERs）通常用于遇到未预见的异常时，保证系统被多个AppDomain或者进程共享的状态的正确性。</p>
<p>这种异常我们通常称之为Asynchronous Exception。比如当调用一个函数时，CLR需要去加载assembly，在AppDomain的堆上创建类型，调用类型的类构造函数，JIT把IL转换成native代码等等。当这些过程出错时，CLR会抛异常。如果这个异常是在代码的catch或者finally抛出的话，catch和finally中的错误恢复代码就不能被执行了，这样系统的状态就有可能会出错。</p>
<p>考虑如下的代码示例：</p>


```c#
sealed class Type1
    {
        static Type1()
        {
            // throw new ApplicationException("if this throws an exception, M won’t get called");
            Console.WriteLine("Type1's static ctor called");
        }

        [ReliabilityContract(Consistency.WillNotCorruptState, Cer.Success)]
        public static void M()
        {
            Console.WriteLine("Type1's M");
        }
    }

        static void CER()
        {
            RuntimeHelpers.PrepareConstrainedRegions();
            try
            {
                Console.WriteLine("In try");
            }
            finally
            {
                // Type1’s static constructor is implicitly called in here
                Type1.M();
            }
        }
```

<p>第9行和第18行使用了CER，需要<span style="color: #0000ff;">using</span><span style="color: #000000;"> System.Runtime.CompilerServices; 和</span><span style="color: #0000ff;">using</span> System.Runtime.ConstrainedExecution;。当JIT编译器看到CERs时，会先编译catch和finally中的代码。JIT会加载需要的assembly，创建对象，调用类构造函数，JIT所有的方法。如果任何一步出了异常，那么这个异常会在try代码块之前抛出，从而保证了系统的状态始终是正确的。<span style="color: blue;"><br /></span></p>
<p>运行上面的代码，可以得到如下的输出：</p>

```
Type1's static ctor called
In try
Type1's M
```
<p>&nbsp;如果注释掉第9行和第18行，得到的输出如下：</p>

```
In try
Type1's static ctor called
Type1's M
```

补充几个关于Exception的小知识：

1. 在watch窗口输入$exception，可以看到CLR的exception信息。对于C++，可以在watch窗口输入@err,hr，相当于显示上一次调用API后再调用GetLastError。  
1. 输入@eax，hr显示eax寄存器的值，由于win的API的返回值放在eax中，所以这句话的意思就是的到最近一个API的返回值。  
1. 如果你需要看的是一个array，你可以通过在watch中写 array, 3来看数组的前3个值。  
1. 可以监听AppDomain的FirstChanceException事件来找到exception的第一现场。  
1. 如果想先catch在throw。应该用`catch(e){throw;} `而不是 `catch(e){throw e;}`。  
1. 在函数前加属性`[MethodImpl](MethodImplOptions.NoInlining)] `可以强迫JIT在异常抛出时不要inline这个方法。我在之前的博客中介绍过如何使用debugger attribute来定制在Visual Studio中的信息。参见[定制自己的Visual Studio的Debugger Visualizer](/2012/07/16/customize-your-own-debugger-visualizer-in-csharp/)和[定制C#在Visual Studio中的debug信息](http://www.cnblogs.com/fresky/articles/2133378.html)  
1. 可以调用 `SetErrorMode(SEM_NOGPFAULTERRORBOX)` 来禁用Windows Error Reporting，就是出错后问你要不要发送出错信息的那个对话框。
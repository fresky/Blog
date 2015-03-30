title: 用Windbg来看看CLR的JIT是什么时候发生的
date: 2015-03-28 19:09:23
categories:
tags: [CSharp, Debug]
description: 本文用Windbg来调试看C#的程序在运行时JIT发生在什么时候。
---
C#的程序一个可能遇到的性能问题就是启动时间可能会太长，这是因为C#编译后会成为IL，在第一次运行时会由JIT即时编译编译为机器代码，然后运行。[.NET Just in Time Compilation and Warming up Your System](http://blogs.msdn.com/b/abhinaba/archive/2014/09/29/net-just-in-time-compilation-and-warming-up-your-system.aspx)介绍了作者对于JIT的一些经验，其中谈到了如何通过Windbg来看JIT究竟是什么时候发生的，很有意思。下面我们也就拿段小程序用Windbg看看。

假设有如下一个简单的程序，编译成Demo.exe。

```c#
namespace Demo
{
    class MyClass
    {
        public void a()
        {
            int i = 0;
            while (true)
            {
                b(i++);
            }
        }

        public void b(int i)
        {
            Console.WriteLine(i);
        }
    }

    class Program
    {

        static void Main(string[] args)
        {
            MyClass mc = new MyClass();
            mc.a();
        }
    }
}
```

然后我们打开Windbg，按快捷键`Ctrl+E`，或者点击File->Open Executable，然后选择刚才编译好的Demo.exe。这样就用Windbg把Demo.exe运行起来了。

接着在Windbg里运行下面的命令来让Windbg在Demo.exe加载clr时停下来。

```
sxe ld:clr
```

然后输入`g`命令，很快Windbg就会停下来，这时敲`k`命令可以看到如下的输出，正在加载clr。

```
0:000> k
ChildEBP RetAddr  
0018f6fc 7796bf70 ntdll!ZwMapViewOfSection+0x12
0018f750 7796c5fb ntdll!LdrpMapViewOfSection+0xc7
0018f844 7796c42c ntdll!LdrpFindOrMapDll+0x333
0018f9c4 7796c558 ntdll!LdrpLoadDll+0x2b2
0018f9fc 755b2ca2 ntdll!LdrLoadDll+0xaa
0018fa38 74045ba3 KERNELBASE!LoadLibraryExW+0x1f1
0018fb9c 7404b32c mscoreei!RuntimeDesc::LoadLibrary+0xcf
0018fbd8 740468f7 mscoreei!RuntimeDesc::LoadMainRuntimeModuleHelper+0x96
0018fc1c 7404a70a mscoreei!RuntimeDesc::LoadMainRuntimeModule+0x1b8
0018fc74 7404a877 mscoreei!RuntimeDesc::EnsureLoaded+0x8e
0018fc8c 7404a960 mscoreei!RuntimeDesc::GetProcAddressInternal+0xe
0018fce0 7404ff19 mscoreei!CLRRuntimeInfoImpl::GetProcAddress+0x48
0018fd28 7404ffaf mscoreei!GetCorExeMainEntrypoint+0xe9
0018fd68 74267f16 mscoreei!_CorExeMain+0x54
0018fd78 74264de3 MSCOREE!ShellShim__CorExeMain+0x99
0018fd80 7544338a MSCOREE!_CorExeMain_Exported+0x8
0018fd8c 77969f72 KERNEL32!BaseThreadInitThunk+0xe
0018fdcc 77969f45 ntdll!__RtlUserThreadStart+0x70
0018fde4 00000000 ntdll!_RtlUserThreadStart+0x1b
```

这时我们在Windbg里运行命令`.loadby sos clr`来加载sos，然后运行命令`!bpmd Demo.exe Demo.Program.Main`在Main函数上加一个断点，然后运行命令`g`继续，一会儿就会停在Main函数上。这个时候再运行`!bpmd Demo.exe Demo.MyClass.a`加一个断点在函数a上，运行命令`g`继续，等停下来之后运行命令`!dso`来看看当前都有什么对象，命令输出如下：

```
0:000> !dso
OS Thread Id: 0x1978 (0)
ESP/REG  Object   Name
ecx      020f230c Demo.MyClass
0018F2C4 020f230c Demo.MyClass
0018F2DC 020f230c Demo.MyClass
0018F2E0 020f230c Demo.MyClass
0018F2E4 020f22fc System.Object[]    (System.String[])
0018F360 020f22fc System.Object[]    (System.String[])
0018F4BC 020f22fc System.Object[]    (System.String[])
0018F4EC 020f22fc System.Object[]    (System.String[])
0018FA20 020f1238 System.SharedStatics
```

找到我们需要的`Demo.MyClass`的地址`020f230c`，运行命令`!do 020f230c`来找到`Demo.MyClass`的MethodTable，输出如下：

```
0:000> !do 020f230c 
Name:        Demo.MyClass
MethodTable: 00293898
EEClass:     002914a0
Size:        12(0xc) bytes
File:        D:\Documents\Visual Studio 2013\Projects\Demo\bin\Debug\Demo.exe
Fields:
None
```

运行命令`!dumpmt -md 00293898`来看看`Demo.MyClass`的函数，输出如下：

```
0:000> !dumpmt -md 00293898
EEClass:         002914a0
Module:          00292edc
Name:            Demo.MyClass
mdToken:         02000002
File:            D:\Documents\Visual Studio 2013\Projects\Demo\bin\Debug\Demo.exe
BaseSize:        0xc
ComponentSize:   0x0
Slots in VTable: 7
Number of IFaces in IFaceMap: 0
--------------------------------------
MethodDesc Table
   Entry MethodDe    JIT Name
7290952c 7262612c PreJIT System.Object.ToString()
7291ec30 72626134 PreJIT System.Object.Equals(System.Object)
7291e860 72626154 PreJIT System.Object.GetHashCode()
7291e2a0 72626168 PreJIT System.Object.Finalize()
004800b0 00293890    JIT Demo.MyClass..ctor()
004800e8 00293878    JIT Demo.MyClass.a()
0029c041 00293884   NONE Demo.MyClass.b(Int32)
```

这里我们可以清楚地看到`Demo.MyClass.a`已经被JIT了，而`Demo.MyClass.b`还没有。所以**JIT的粒度是函数级的**。

**所以我们在写C#的代码时要尽量做到模块化，特别是有一些很少会用到的代码，比如异常处理和日志，一定不要直接放在Main函数里，把他们抽到别的函数里就可以保证在需要他们的时候才JIT，从而不会影响到启动性能。**


我们也可以直接在JIT一个方法的入口函数`clr!UnsafeJitFunction`上加个断点，`bp clr!UnsafeJitFunction`，然后继续运行`g`，等断点出发后我们看看callstack，运行`!dumpstack`，输出如下：

```
0:000> !dumpstack
OS Thread Id: 0x1978 (0)
Current frame: clr!UnsafeJitFunction
ChildEBP RetAddr  Caller, Callee
0018f0d0 739d878c clr!MethodDesc::MakeJitWorker+0x36e, calling clr!UnsafeJitFunction
0018f0f8 77963cfe ntdll!RtlAllocateHeap+0x23a, calling ntdll!RtlpAllocateHeap
0018f1ac 73ae97b5 clr!MethodDesc::DoPrestub+0x598, calling clr!MethodDesc::MakeJitWorker
0018f1f8 739ba4e5 clr!ETWTraceStartup::ETWTraceStartup+0x37, calling clr!ETWTraceStartup::StartupTraceEvent
0018f208 739ba33b clr!MethodDesc::CheckRestore+0x23, calling clr!MethodTable::IsFullyLoaded
0018f224 739bb02b clr!PreStubWorker+0xf0, calling clr!MethodDesc::DoPrestub
0018f28c 739a2a0c clr!ThePreStub+0x16, calling clr!PreStubWorker
0018f2bc 0048012f (MethodDesc 00293878 +0x47 Demo.MyClass.a()), calling 0029c041
0018f2d4 00480099 (MethodDesc 002937e0 +0x49 Demo.Program.Main(System.String[])), calling (MethodDesc 00293878 +0 Demo.MyClass.a())
```

从这里我们可以看到**JIT发生在函数调用的同一个线程里**。


另外，提高系统启动性能通常的做法有
1. [Ngen.exe (Native Image Generator)](https://msdn.microsoft.com/en-us/library/6t9t5wcf%28v=vs.110%29.aspx)，参见我的博文[使用MPGO和NGEN来优化C#桌面程序的启动性能](/2012/12/18/using-mpgo-and-ngen-to-optimize-csharp-app-starting-performance/)。

2. 强制JIT，下面是一段示例代码来通过[RuntimeHelpers.PrepareMethod](http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.runtimehelpers.preparemethod(v=vs.110).aspx)API强制JIT。

```
using System;
using System.Reflection;
namespace Demo
{
    class Program
    {
        static private void ForceJit(Assembly assembly)
        {
            var types = assembly.GetTypes();

            foreach (Type type in types)
            {
                var ctors = type.GetConstructors(BindingFlags.NonPublic
                                            | BindingFlags.Public
                                            | BindingFlags.Instance
                                            | BindingFlags.Static);

                foreach (var ctor in ctors)
                {
                    JitMethod(assembly, ctor);
                }

                var methods = type.GetMethods(BindingFlags.DeclaredOnly
                                       | BindingFlags.NonPublic
                                       | BindingFlags.Public
                                       | BindingFlags.Instance
                                       | BindingFlags.Static);
                
                foreach (var method in methods)
                {
                    JitMethod(assembly, method);
                }
            }
        }

        static private void JitMethod(Assembly assembly, MethodBase method)
        {
            if (method.IsAbstract || method.ContainsGenericParameters)
            {
                return;
            }
            
            System.Runtime.CompilerServices.RuntimeHelpers.PrepareMethod(method.MethodHandle);
        }

        static void Main(string[] args)
        {
            ForceJit(Assembly.LoadFile(@"d:\Scratch\asm.dll"));
        }
    }
}
```

小结一下本文用到的Windbg命令。

```
sxe ld:clr                        // 当加载clr时break
!bpmd Demo.exe Demo.Program.Main  // 在Main函数上加断点
!dumpmt -md                       // 看类的函数具体信息，有没有被JIT
bp clr!UnsafeJitFunction          // 在JIT函数入口加断点
```

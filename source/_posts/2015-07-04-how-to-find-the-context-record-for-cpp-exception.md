title: 如何用Windbg找到被catch住的C++的异常
date: 2015-07-04 17:39:11
categories:
tags: [Debug, CPP]
description: 本文介绍了如果用windbg的搜索内存的命令s [-[[Flags]Type]] Range Pattern在栈上寻找上下文CONTEXT，然后通过windbg的切换上下文的命令.cxr来找到C++的异常的调用栈（call stack）信息。
---
在调试dump时经常会遇到我们找到的异常并不是第一个异常，而是catch别的异常之后又抛出来的。在这种情况下，我们需要找到第一个异常来确定问题究竟出在那里。

如果是C#的异常，通常比较简单，在我们运行SOS命令`!pe`之后，会列出`InnerException`，然后直接在`!pe innerexceptionaddress`就好了。

如果是C++的异常，就没有这么直接了，那怎么能找到C++的Inner Exception呢？

# Windows的CONTEXT定义
有一个小技巧可以很方便的找到异常。这个技巧基于操作系统的一个特性。当Windows操作系统抛出异常时，它会把抛出异常的[上下文（CONTEXT）](https://msdn.microsoft.com/en-us/library/windows/desktop/ms679284%28v=vs.85%29.aspx)放进栈里面。所以只要我们能够找到这个CONTEXT，我们就能找到异常的调用栈（call stack）了。

这个CONTEXT的定义可以在`winnt.h`中找到，我们也可以直接在Windbg中看看。
```
0:048> dt nt!_context
ntdll!_CONTEXT
   +0x000 ContextFlags     : Uint4B
   +0x004 Dr0              : Uint4B
   +0x008 Dr1              : Uint4B
   +0x00c Dr2              : Uint4B
   +0x010 Dr3              : Uint4B
   +0x014 Dr6              : Uint4B
   +0x018 Dr7              : Uint4B
   +0x01c FloatSave        : _FLOATING_SAVE_AREA
   +0x08c SegGs            : Uint4B
   +0x090 SegFs            : Uint4B
   +0x094 SegEs            : Uint4B
   +0x098 SegDs            : Uint4B
   +0x09c Edi              : Uint4B
   +0x0a0 Esi              : Uint4B
   +0x0a4 Ebx              : Uint4B
   +0x0a8 Edx              : Uint4B
   +0x0ac Ecx              : Uint4B
   +0x0b0 Eax              : Uint4B
   +0x0b4 Ebp              : Uint4B
   +0x0b8 Eip              : Uint4B
   +0x0bc SegCs            : Uint4B
   +0x0c0 EFlags           : Uint4B
   +0x0c4 Esp              : Uint4B
   +0x0c8 SegSs            : Uint4B
   +0x0cc ExtendedRegisters : [512] UChar
```
从上面我们可以看到`_CONTEXT`的开头是`ContextFlags`，它的定义也可以在`winnt.h`中找到，对于32位Windows来说，它的值通常是`CONTEXT_ALL`，就是`0x1003f`。`winnt.h`中的定义如下。

```cpp
#define CONTEXT_i386    0x00010000L    // this assumes that i386 and
#define CONTEXT_i486    0x00010000L    // i486 have identical context records

// end_wx86

#define CONTEXT_CONTROL         (CONTEXT_i386 | 0x00000001L) // SS:SP, CS:IP, FLAGS, BP
#define CONTEXT_INTEGER         (CONTEXT_i386 | 0x00000002L) // AX, BX, CX, DX, SI, DI
#define CONTEXT_SEGMENTS        (CONTEXT_i386 | 0x00000004L) // DS, ES, FS, GS
#define CONTEXT_FLOATING_POINT  (CONTEXT_i386 | 0x00000008L) // 387 state
#define CONTEXT_DEBUG_REGISTERS (CONTEXT_i386 | 0x00000010L) // DB 0-3,6,7
#define CONTEXT_EXTENDED_REGISTERS  (CONTEXT_i386 | 0x00000020L) // cpu specific extensions

#define CONTEXT_FULL (CONTEXT_CONTROL | CONTEXT_INTEGER |\
                      CONTEXT_SEGMENTS)

#define CONTEXT_ALL             (CONTEXT_CONTROL | CONTEXT_INTEGER | CONTEXT_SEGMENTS | \
                                 CONTEXT_FLOATING_POINT | CONTEXT_DEBUG_REGISTERS | \
                                 CONTEXT_EXTENDED_REGISTERS)

#define CONTEXT_XSTATE          (CONTEXT_i386 | 0x00000040L)
```

# 在栈上搜索`0x1003f`来找CONTEXT
Windbg有一个`s`的命令用来搜索内存，它的格式为`s [-[[Flags]Type]] Range Pattern`。通过CONTEXT的定义我们知道了Pattern就是`CONTEXT_ALL`（也就是`0x1003f`），那怎么给出当前栈的Range呢。

Windbg中可以用`!teb`来看线程环境块（Thread Environment Block），输出如下。里面包含栈的开始StackBase和结束地址StackLimit。
```
0:048> !teb
TEB at fff23000
    ExceptionList:        296ae06c
    StackBase:            296b0000
    StackLimit:           296a9000
    SubSystemTib:         00000000
    FiberData:            00001e00
    ArbitraryUserPointer: 00000000
    Self:                 fff23000
    EnvironmentPointer:   00000000
    ClientId:             00001dcc . 000009d0
    RpcHandle:            00000000
    Tls Storage:          0cb43f80
    PEB Address:          fffde000
    LastErrorValue:       0
    LastStatusValue:      0
    Count Owned Locks:    0
    HardErrorMode:        0
```

这样我们就能用下面的命令在栈上搜索CONTEXT了。
```
s -d poi(@$teb+0x8)  poi(@$teb+0x4) 0x1003f 
```

搜索的结果类似如下：
```
0:048> s -d poi(@$teb+0x8)  poi(@$teb+0x4) 0x1003f
296ae428  0001007f 00000000 00000000 00000000  ................
```

这个时候我们只要运行如下命令就能得到这个CONTEXT对应的函数调用栈（call stack）了。
```
.cxr 296ae428
kbn
```

# 在栈上搜索`@gs @fs @es @ds`寄存器来找CONTEXT
但是有时候`ContextFlags`不是`CONTEXT_ALL`（也就是`0x1003f`）。我就遇到过是`CONTEXT_XSTATE`（也就是`0x1007f`）的情况，所以假如你用上面的搜索命令没有找到任何输出，试试下面这个。
```
s -d poi(@$teb+0x8)  poi(@$teb+0x4) 0x1007f
```

那有没有更好的办法能肯定找到CONTEXT呢？我们再去看看`ntdll!_CONTEXT`的结构吧。我们发现在从`0x8c`开始，它存放了4个寄存器`gs`，`fs`，`es`和`ds`的地址。
```
   +0x08c SegGs            : Uint4B
   +0x090 SegFs            : Uint4B
   +0x094 SegEs            : Uint4B
   +0x098 SegDs            : Uint4B
```

所以我们可以用这四个寄存器地址来搜索了，注意这个搜索到的结果需要`-0x8c`才是CONTEXT的地址。
```
s -d poi(@$teb+0x8) poi(@$teb+0x4) @gs @fs @es @ds
```

# 用`.foreach`让这个过程自动化

也可以结合`.foreach`命令，自动对每一个搜索到的CONTEXT地址运行`.cxr`和`kbn`的命令，如下所示：

```
.foreach (context {s -[w1]d poi(@$teb+0x8) poi(@$teb+0x4) @gs @fs @es @ds}) {.cxr context - 8c; kbn}
```
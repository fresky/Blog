title: 调试内存泄漏问题的一些经验
date: 2015-06-21 14:03:45
categories:
tags: [Debug, CPP, CSharp]
description: 内存泄漏（memory leak）是软件中经常遇到的一类问题，这类问题又是比较难以检测的，通常我们在程序遇到Out Of Memory的异常时才会注意到。拿到Out Of Memory的dump文件后，如何分析dump文件找到内存泄漏的线索又是一个难点。这篇文章分享了一些在Windows平台如何调试，检测C++和C#的内存泄漏的一些经验。Windbg，VLD，gflags，UMDH，Visual Studio。
---
内存泄漏（memory leak）是软件中经常遇到的一类问题，这类问题又是比较难以检测的，通常我们在程序遇到`Out Of Memory`的异常时才会注意到。拿到`Out Of Memory`的dump文件后，如何分析dump文件找到内存泄漏的线索又是一个难点。这篇文章分享了一些在Windows平台如何调试，检测C++和C#的内存泄漏的一些经验。

#一、内存泄漏的Dump分析
通常拿到`Out Of Memory`的dump之后，用windbg打开，抛出异常的call stack一般是在分配内存时，这个call stack其实意义不大，我们需要知道的是内存为什么被用完了。

一般第一个要运行的命令是`!address -summary`，它会给出一个内存情况的总结，如下所示。
```
0:000> !address -summary

--- Usage Summary ---------------- RgnCount ----------- Total Size -------- %ofBusy %ofTotal
Heap                                    340          3c876000 ( 968.461 Mb)  48.38%   47.29%
<unknown>                              2001          2c3b5000 ( 707.707 Mb)  35.36%   34.56%
Image                                  1115          119cf000 ( 281.809 Mb)  14.08%   13.76%
Free                                    544           2e40000 (  46.250 Mb)            2.26%
Stack                                   139           2b00000 (  43.000 Mb)   2.15%    2.10%
Other                                    95             8a000 ( 552.000 kb)   0.03%    0.03%
TEB                                      43             2b000 ( 172.000 kb)   0.01%    0.01%
PEB                                       1              1000 (   4.000 kb)   0.00%    0.00%

--- Type Summary (for busy) ------ RgnCount ----------- Total Size -------- %ofBusy %ofTotal
MEM_PRIVATE                             889          46725000 (   1.101 Gb)  56.31%   55.04%
MEM_MAPPED                             1686          24707000 ( 583.027 Mb)  29.13%   28.47%
MEM_IMAGE                              1159          12384000 ( 291.516 Mb)  14.56%   14.23%

--- State Summary ---------------- RgnCount ----------- Total Size -------- %ofBusy %ofTotal
MEM_COMMIT                             3425          717c1000 (   1.773 Gb)  90.71%   88.66%
MEM_RESERVE                             309           b9ef000 ( 185.934 Mb)   9.29%    9.08%
MEM_FREE                                544           2e40000 (  46.250 Mb)            2.26%

--- Protect Summary (for commit) - RgnCount ----------- Total Size -------- %ofBusy %ofTotal
PAGE_READWRITE                         2300          5e7fd000 (   1.477 Gb)  75.54%   73.83%
PAGE_EXECUTE_READ                       201           abb4000 ( 171.703 Mb)   8.58%    8.38%
PAGE_READONLY                           656           616f000 (  97.434 Mb)   4.87%    4.76%
PAGE_WRITECOPY                          125           20d3000 (  32.824 Mb)   1.64%    1.60%
PAGE_READWRITE|PAGE_GUARD                86             bb000 ( 748.000 kb)   0.04%    0.04%
PAGE_EXECUTE_READWRITE                   44             a4000 ( 656.000 kb)   0.03%    0.03%
PAGE_EXECUTE_WRITECOPY                   13             6f000 ( 444.000 kb)   0.02%    0.02%

--- Largest Region by Usage ----------- Base Address -------- Region Size ----------
Heap                                         7f30000            fd0000 (  15.813 Mb)
<unknown>                                   12670000           4020000 (  64.125 Mb)
Image                                       68f27000           19cb000 (  25.793 Mb)
Free                                        717f2000             7e000 ( 504.000 kb)
Stack                                        2cf0000             fd000 (1012.000 kb)
Other                                       7efb0000             23000 ( 140.000 kb)
TEB                                         7eeb0000              1000 (   4.000 kb)
PEB                                         7efde000              1000 (   4.000 kb)
```

第一个部分是使用情况，按照大小排序。通常排第一的不是`Heap`就是`<unkown>`。`Heap`是C++的非托管内存，`<unkown>`是C#的托管内存。

第二个部分是类型情况，分了3类，分别是：  
1. `MEM_PRIVATE`：当前进程独占的内存。  
1. `MEM_MAPPED`：映射到文件的内存，这些文件不属于进程程序本身，比如Memory Mapping File。  
1. `MEM_IMAGE`：映射到进程程序的内存，比如程序加载的dll。  

最后一个部分是最大连续内存，比如上图中我们可以看到现在最大的连续可用内存只有500k了。

##非托管（C++）内存泄漏分析
如果`!address -summary`的输出中发现`Heap`被用掉了很多，那很有可能有C++的内存泄漏，我们需要检查堆（heap）来找到可疑的对象。

1.通过`!heap -s`来看堆的使用情况，会把堆按照大小列出来，如下所示：

```
0:000> !heap -s
LFH Key                   : 0x352f041c
Termination on corruption : DISABLED
  Heap     Flags   Reserv  Commit  Virt   Free  List   UCR  Virt  Lock  Fast 
                    (k)     (k)    (k)     (k) length      blocks cont. heap 
-----------------------------------------------------------------------------
Virtual block: 39ee0000 - 39ee0000 (size 00000000)
00520000 00000002  619112 617928 619112   1712  2165    95    1      0   LFH
00ef0000 00001002    1088    324   1088      6     5     2    0      0   LFH
02590000 00001002    1088    316   1088     12     7     2    0      0   LFH
02aa0000 00001002    1088    256   1088      4     4     2    0      0   LFH
00340000 00001002      64     12     64      1     2     1    0      0 
```
2.我们用`!heap -stat -h <HeapEntry>`来看最大的那个堆，它会列出这个堆上分配的所有对象的统计情况。如果幸运的话我们会看到某个大小的对象数目非常大，占用了很多内存。比如下面的例子中大小为`1f64`的对象有`0x76c6`个，占了99%的内存。
  
```
0:000> !heap -stat -h 00520000
 heap @ 00520000
group-by: TOTSIZE max-display: 20
    size     #blocks     total     ( %) (percent of total busy bytes)
    1f64 76c6 - e905f58  (99.99)
    1800 1 - 1800  (0.00)
    824 2 - 1048  (0.00)
```
3.我们可以用`!heap -flt s <ObjSize>`来列出所有大小是制定大小的对象。输出结果中的`UserPtr`就是对象的地址，然后可以用`d`命令来显示这个地址的内容。如果幸运的话，比如这个地址直接存了个字符串，那就好办多了。  
4.还有一些情况我们可能能猜到是某些对象泄漏了。比如如果在`!address -summary`的输出里我们看到`MEM_MAPPED`大的离谱，而我们程序里所有的MemoryMappingFile都继承自某个基类，那么我们就可以直接看看内存中有多少个这类对象。  
5.用命令`x modulename!*classname*table*`来找内存中虚表的地址。  
6.用命令`!heap -srch vtableaddress`来找到所有的对象。  
7.用命令`dt modulename!classname objectaddress`来看对象的内容是什么，接着就能分析出为什么这些对象有这么多。  

##托管（C#）内存泄漏分析
如果`!address -summary`的输出中发现`<unkown>`被用掉了很多，那很有可能有C#的内存泄漏，调试相对简单。

1.运行`loadby sos mscorwks`（.net4之前）或者`loadby sos clr`（.net4及以后）来加载SOS扩展。  
2.运行`!dumpheap -stat`来看托管堆的统计信息，输出如下：

```
total 976456 objects
Statistics:
      MT    Count    TotalSize Class Name
71497de4        1           12 System.Runtime.Remoting.Channels.Tcp.TcpClientTransportSinkProvider
...
```输出是按照TotalSize的递增顺序显示的，直接翻到最后一行，看看是哪个对象占用了最大的TotalSize。    
3.运行`!dumpheap -mt <mt>`来把内存中这个Method Table的所有对象都列出来。结果的第一列就是对象的地址。  
4.运行`!do <address>`来看这个对象的内容是什么。  
5.运行`!gcroot <address>`来看这些对象是被谁引用的，这样多半就能找到发生内存泄漏的原因了。  

##GDI句柄超过限制
还有一种发生`Out Of Memory`异常的情况是GDI句柄超过限制了，可以看到dump中crash的call stack中是有关句柄操作的。默认情况下Windows的每个进程的GDI句柄额度是10000，可以通过注册表`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\WindowsNT\CurrentVersion\Windows\GDIProcessHandleQuota`来修改这个值。

这种情况相对比较好处理，windows上的[GDI对象](https://msdn.microsoft.com/en-us/library/windows/desktop/ms724291%28v=vs.85%29.aspx)就这些：Bitmap，Brush，DC，Enhanced metafile，Enhanced-metafile DC，Font，Memory DC，Metafile，Metafile DC，Palette，Pen and extended pen，Region。

#二、内存泄漏的实时调试
如果可以非常容易重现的话，可以实时调试内存泄漏，这样就会容易很多了。

##非托管内存泄漏检测
###使用VLD来检测内存泄漏。
[VLD](https://vld.codeplex.com/)是一个VC++的开源内存泄漏检测工具，非常易于使用。在调试器中运行程序，会在结束时生成一个内存泄漏的报告，包含内存分配的call stack。
###打开“Create user mode stack trace database”，分析dump
可以用gflags打开“Create user mode stack trace database”，如下所示，这样就会记录下来每个对象创建的call stack，可以就可以很容易的查到泄漏对象是怎么创建出来的了。
{% limg gflags_stack.png %}

####使用Windbg的`!heap -l`命令。
1. 收集dump，用Windbg打开，然后运行命令`.logopen d:\leak.txt`打开log。  
1. 运行`!heap -l`命令，会把所有泄漏的对象列出来，附带创建的call stack。可以很容易的写个程序来分析这个输出，合并重复的对象，计算总大小。  

####使用Windbg的`!heap -p -a <address>`命令
按照上面提到的非托管（C++）内存泄漏分析方法来分析dump，最后找到可以的对象时可以直接运行`!heap -p -a <address>`命令来看到这个地址的对象的创建call stack。

####使用UMDH
[UMDH](https://msdn.microsoft.com/en-us/library/ff558947%28v=vs.85%29.aspx)是Windows Debugging Tools里的，和Windbg在同一个目录里，可以用UMDH收集多个内存的log，然后比较，找出泄漏的对象。

##托管（C#）内存泄漏检测

Visual Studio 2013加入了[调试托管内存](https://msdn.microsoft.com/en-us/library/dn342825.aspx)的功能，在打开dump文件后可以选择"Debug Managed Memory"，可以看到托管对象的大小，数目，root等信息。

![Debug Managed Memory](https://i-msdn.sec.s-msft.com/dynimg/IC720150.png)

![Managed Object List](https://i-msdn.sec.s-msft.com/dynimg/IC750722.png)

也可打开多个dump通过选择“Select baseline”进行内存比较，可以看到内存的变化。

![Compare Managed Memory](https://i-msdn.sec.s-msft.com/dynimg/IC720152.png)

##GDI句柄监测
[GDIView](http://www.nirsoft.net/utils/gdi_handles.html)是一个免费的小工具，可以监测GDI使用情况。
---
layout: post
title: ".net 4中的pInvokeStackImbalance MDA默认是开启的"
date: 2013-07-04
comments: true
tags: CSharp
---
<p>今天把我之前发的一个小工具<a href="/2012/09/10/freeverything-simple-disk-cleanup-tool-based-on-everything/">FreeEverything（基于everything的一个简易磁盘清理工具）</a>升级到了.net framework 4.5，并且去掉了对mvvmlight的依赖。结果在测试运行的时候发现如果用visual studio调试运行，就会触发一个break，显示如下的错误信息。</p>

```
A call to PInvoke function 'SampleMethod' has unbalanced the stack. 
This is likely because the managed PInvoke signature does not match 
the unmanaged target signature. Check that the calling convention and 
parameters of the PInvoke signature match the target unmanaged signature.
```

<p>但是在之前的3.5上就没有问题，报错的地方在下面的语句。</p>

```csharp
EverythingWraper.Everything_Query();
```

<p>于是就调查了一下，发现在<a href="http://msdn.microsoft.com/en-us/library/0htdy0k3.aspx">pInvokeStackImbalance MDA</a>（managed debugging assistant）在.net 3.5中是默认关闭的，如果开启需要到debug-&gt;exception-&gt;mda来打开。但是在.net 4中默认改成了开启，这就解释了为啥在3.5上没问题，但是在4.5上有问题。</p>
<p>如果dllimport时C#中的函数签名和非托管的dll中的函数签名不一致比如参数数目，或者参数大小不对，或者calling convention不对，就会触发visual studio报这个错。</p>
<p>&nbsp;</p>
<p>但是为什么会出这个错误呢？我的dllimport是从<a href="http://support.voidtools.com/everything/Main_Page"> Everything SDK Wiki</a>上的示例代码考过来的，如下所示：</p>

```csharp
        [DllImport(@"ThirdParty\Everything.dll" )]
        public static extern bool Everything_Query();
```

<p>于是我就又打开了Everything的SDK，看了看源代码，发现其实Everything_Query()是有一个参数的，因为参数不匹配，所以visual studio就报了这个错。修改成如下就没问题了。</p>

```csharp
        [DllImport(@"ThirdParty\Everything.dll" )]
        public static extern bool Everything_Query(bool bWait);
```
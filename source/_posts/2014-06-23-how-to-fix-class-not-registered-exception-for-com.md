---
layout: post
title: "遇到Class not registered的COM异常怎么办"
date: 2014-06-23 17:13
comments: true
tags: [CPP,Debug]
keywords: class not registered, com, gflags
description: 本文总结了3种可能导致COM的Class not registered的异常的情况，并且分别给出了解决办法。
---
我通常遇到Class not registered的异常有如下3种原因：

  1. 真的是COM注册有问题。
  2. COM对象的构造函数或者FinalConstruct有问题。
  3. COM对象所在的DLL没有被正确的加载进来。
  

一般遇到Class not registered的异常，会写一个如下的C++小程序，只是调用`CoCreateInstance`，同时在COM对象的构造函数里打个断点，然后跑跑看返回的`HRESULT`是什么。

```cpp
#include "stdafx.h"

int _tmain(int argc, _TCHAR* argv[])
{
	::CoInitialize(NULL);
	CComPtr<IUnknown> comObject;
	HRESULT hr = comObject.CoCreateInstance(L"Progid");
	::CoUninitialize();
	return 0;
}
```

如果是上面说的第一个种错误，那么COM对象的构造函数的断点进不去。这种比较好修复，直接运行`regsvr32 target.dll`就好了。

如果是上面说的第二种错误，那么COM对象的构造函数能够进去，这个时候如果打开exception的话，应该可以看到构造函数或者FinalConstruct里抛出了什么异常，修改这个异常就好了。

如果是上面说的第三种错误，那么这个C++小程序可以正确的运行，并且返回S_OK，这就有点郁闷了。我遇到的例子是在crash之后用windbg打开dump，运行`lm`命令，可以看到这个COM对象所在的module没有被加载，如下所示：
```
Unloaded modules:
???????? ????????   mydll.dll
```

如果dll没有正确的被加载，也是会抛出这个Class not registered的异常的。如何进一步验证呢，这个就又要求助于我在上篇文章[消失的进程——谁是凶手?](http://fresky.github.io/blog/2014/06/20/using-gflags-to-catch-dump-for-the-disappeared-process/)中提到的gflags了，这次需要使用的功能是[Show loader snaps](http://msdn.microsoft.com/en-us/library/windows/hardware/ff556886%28v=vs.85%29.aspx)，如下图所示：

{% limg gflags_loader_snaps.png %}

这样程序运行后，windbg会自动attach上去，然后就可以在windbg的窗口看到所有加载和卸载module的详细信息，比如在我遇到的这个问题里面会发现如下的错误信息：

```
- LdrpGenericExceptionFilter - ERROR: Function LdrpSnapIAT raised exception 0xc0000139
```

0xc0000139的意思是说Entry point not found，通过这些详细信息应该可以定位到为什么dll没有加载成功。
 
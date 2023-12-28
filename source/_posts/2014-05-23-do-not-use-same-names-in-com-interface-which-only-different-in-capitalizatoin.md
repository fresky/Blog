---
layout: post
title: "在COM接口中不要使用同时出现只是大小写不同的名字作为属性名、函数名或者参数名"
date: 2014-05-23 08:49
comments: true
tags: Programming
keywords: 
description: 
---
昨天被一个编译错误折磨了很久，从中学到了这个教训：**在COM接口中不要使用同时出现只是大小写不同的名字作为属性名、函数名或者参数名**。

在一个.h文件中有2个接口如下：
```c++
__interface A : IUnknown
{
    …
    [propget, helpstring("property Name")]
    HRESULT Name( [out, retval] BSTR* pName);
	…
};

__interface B : IUnknown
{
    …
};
```

我在接口B中新加了一个方法如下：
```c++
__interface B : IUnknown
{
    …
    [helpstring("method SetName")]
    HRESULT SetName([in] BSTR name);
	…
};
```

结果编译别的使用A的工程时遇到了如下的错误：
```bash
1> error C2039: 'get_Name' : is not a member of 'ATL::_NoAddRefReleaseOnCComPtr<T>'
1>          with
1>          [
1>              T=A
1>          ]

```

打开对应.tlb后发现，接口A的方法变成了**get_name**。应该是**get_Name**。

从MSDN的[COM Technical Overview](http://msdn.microsoft.com/en-us/library/windows/desktop/ff637359%28v=vs.85%29.aspx)可以找到如下关于命名的说明：

> Names are **stored with case preserved**, but they are **compared as case-insensitive**. Applications that define storage element names must choose names that work in either situation.

就是说当Visual Studio生成.tlb文件的时候，认为属性名Name和参数名name是一样的。所以生成get方法时就使用了小写的n。

如果把新加的B的接口改成如下就没有错误了。
```c++
__interface B : IUnknown
{
    …
    [helpstring("method SetName")]
    HRESULT SetName([in] BSTR paraName);
	…
};
```
所以：**在COM接口中不要使用同时出现只是大小写不同的名字作为属性名、函数名或者参数名**。
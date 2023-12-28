---
layout: post
title: "怎么看C++对象的内存结构 和 怎么解密C++的name mangling"
date: 2012-12-23
comments: true
tags: Programming
---
<p><a href="http://eli.thegreenplace.net/2012/12/17/dumping-a-c-objects-memory-layout-with-clang/">Dumping a C++ object’s memory layout with Clang</a>这篇文章讲述了怎么用Clang来看C++对象的结构，回复中Marek提到了怎么在Visual Studio中看。具体方法如下：</p>  <p>C++项目右键属性，C/C++下的Command Line，加上这个选项</p>  
```
/d1reportAllClassLayout
```

<p>这样在编译时就会在output窗口看到所有的对象的内存结构了。</p>

<p>由于C++编译器会做Name Mangling，我们可以用<strong>undname</strong>这个工具来看到没有被mangling样子。</p>

```
>undname
Microsoft (R) C++ Name Undecorator
Copyright (C) Microsoft Corporation. All rights reserved.

Usage: undname [flags] fname [fname...]
   or: undname [flags] file
```

<p>&#160;</p>

<p>下面是个例子，假设我们有如下的A和B两个类。</p>


```cpp
class A
{
    int a;
public:
    void virtual foo(){};
};

class B:public A
{
    int b;
public:
    void foo(){};
};
```

<p>它们通过<strong>/d1reportAllClassLayout</strong>的结果如下：</p>

```
1>  class A    size(8):
1>      +---
1>   0    | {vfptr}
1>   4    | a
1>      +---
1>  
1>  A::$vftable@:
1>      | &A_meta
1>      |  0
1>   0    | &A::foo
1>  
1>  A::foo this adjustor: 0
1>  
1>  
1>  class B    size(12):
1>      +---
1>      | +--- (base class A)
1>   0    | | {vfptr}
1>   4    | | a
1>      | +---
1>   8    | b
1>      +---
1>  
1>  B::$vftable@:
1>      | &B_meta
1>      |  0
1>   0    | &B::foo
1>  
1>  B::foo this adjustor: 0
```
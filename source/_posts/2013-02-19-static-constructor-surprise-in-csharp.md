---
layout: post
title: "C#的继承类中static constructor的调用问题"
date: 2013-02-19
comments: true
tags: CSharp
---
<p>Eric Lippert写了一系列的关于static constructor的文章，<a href="http://ericlippert.com/2013/02/06/static-constructors-part-one/?utm_source=rss&amp;utm_medium=rss&amp;utm_campaign=static-constructors-part-one">1</a>，<a href="http://ericlippert.com/2013/02/11/static-constructors-part-two/?utm_source=rss&amp;utm_medium=rss&amp;utm_campaign=static-constructors-part-two">2</a>，<a href="http://ericlippert.com/2013/02/14/static-constructors-part-three/?utm_source=rss&amp;utm_medium=rss&amp;utm_campaign=static-constructors-part-three">3</a>，<a href="http://ericlippert.com/2013/02/18/static-constructors-part-four/?utm_source=rss&amp;utm_medium=rss&amp;utm_campaign=static-constructors-part-four">4</a>，可以读读对static constructor有更好的理解。</p>  <p>转一个其中的例子吧，假设有如下代码。</p>  

```csharp
using System;
class B
{
  static B() { Console.WriteLine("B cctor"); }
  public B() { Console.WriteLine("B ctor"); }
  public static void M() { Console.WriteLine("B.M"); }
}
class D : B
{
  static D() { Console.WriteLine("D cctor"); }
  public D() { Console.WriteLine("D ctor"); }
  public static void N() { Console.WriteLine("D.N"); }
}
class P 
{
  static void Main()
  {
    System.Console.WriteLine("Main");
    new D();
  }  
}
```

<p>如果main函数是这样的，</p>

```csharp
static void Main() 
{
  D.M();
}
```

<p>哪个static constructor会被调到呢？</p>

<p>答案是只有B的会被调到。</p>

<p>如果main函数是这样的，</p>

```csharp
static void Main() 
{
  D.N();
}
```

<p>哪个static constructor会被调到呢？</p>

<p>答案是只有D的会被调到。</p>

<p>是不是有点意外：）</p>
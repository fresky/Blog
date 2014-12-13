---
layout: post
title: "C#中静态构造函数导致的一个deadlock"
date: 2013-02-05
comments: true
tags: CSharp
---
<p id="site-description">Eric Lipper的这篇<a href="http://ericlippert.com/2013/01/31/the-no-lock-deadlock/" target="_blank">博客</a>举了一个C#中静态构造函数导致的deadlock的例子，很有意思。</p>
<p>代码如下：</p>

```c#
class C
{
  static C() 
  {
    // Let's run the initialization on another thread!
    var thread = new System.Threading.Thread(Initialize);
    thread.Start();
    thread.Join();
  }
  static void Initialize() { }
  static void Main() { }
}
```

<p>原因很简单，静态构造函数需要在第一次用到这个类静态方法或者实例之前调用结束，就是说C（）在等Initialize（），但是Initialize（）必须在C()结束后才能被调到。</p>
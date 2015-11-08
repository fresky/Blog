---
layout: post
title: "C#的overload检查总是发生在编译时吗？"
date: 2012-10-25
comments: true
tags: CSharp
---
<p><a href="http://stackoverflow.com/questions/12842712/why-does-the-c-sharp-compiler-not-fault-code-where-a-static-method-calls-an-inst?newsletter=1&amp;nlcode=55866|c739">stackoverflow</a>上有人问为什么下面的代码可以编译成功，但是运行时报错：</p>  

```csharp
public sealed class Example
{
    int count;

    public static void Foo( dynamic x )
    {
        Bar(x);
    }

    void Bar( dynamic x )
    {
        count++;
    }
}
```

原因是C#如果参数中有dynamic，或者传进来的值是dynamic时，overload的检查发生在运行时，而不是编译时。<a href="http://msdn.microsoft.com/en-us/library/dd264736.aspx">MSDN</a>上有详细的说明。



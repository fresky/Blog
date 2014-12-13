---
layout: post
title: "C#的static constructor抛了异常会怎么处理？"
date: 2013-02-19
comments: true
tags: CSharp
---
<p><a href="http://stackoverflow.com/questions/4737875/exception-in-static-constructor/4737910#4737910">stackoverflow</a>上举了这个例子说明在C#中，如果static constructor只会被调用一次，即使抛了异常，也不会重试调用。如果抛了异常，那么在这个appdomain里面，这个类就不能用了。</p>  <p>示例代码：</p>  

```c#
using System;

public sealed class Bang
{
    static Bang()
    {
        Console.WriteLine("In static constructor");
        throw new Exception("Bang!");
    }

    public static void Foo() {}
}

class Test
{
    static void Main()
    {
        for (int i = 0; i < 5; i++)
        {
            try
            {
                Bang.Foo();
            }
            catch (Exception e)
            {
                Console.WriteLine(e.GetType().Name);
            }
        }
    }
}
```



<p>输出如下：</p>

```
In static constructor
TypeInitializationException
TypeInitializationException
TypeInitializationException
TypeInitializationException
TypeInitializationException
```
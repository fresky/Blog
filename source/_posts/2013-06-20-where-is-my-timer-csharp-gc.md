---
layout: post
title: "谁动了我的timer？C#的垃圾回收和调试"
date: 2013-06-20
comments: true
tags: CSharp
---
<p>先来看如下的一段代码：</p>  

```c#
using System;
using System.Threading;
public static class Program
{
    public static void Main()
    {
        // Create a Timer object that knows to call our TimerCallback
        // method once every 1000 milliseconds.
        Timer t = new Timer(TimerCallback, null, 0, 1000);
        // Wait for the user to hit <Enter>
        Console.ReadLine();
    }
    private static void TimerCallback(Object o)
    {
        // Display the date/time when this method got called.
        Console.WriteLine("In TimerCallback: " + DateTime.Now);
    } 
}
```



<p>我们在main函数中生成了一个timer，然后这个timer会每隔一秒输出一条记录，显示当前的时间。</p>

<p>如果运行这个程序（release模式），我们可以得到如下的输出：</p>

<p><a href="http://images.cnitblog.com/blog/163228/201306/20111611-5931441e1d014546ac28b1aac12bfee7.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://images.cnitblog.com/blog/163228/201306/20111611-0b9ea3a4bc834dedb860349496c14d07.png" width="535" height="153" /></a></p>

<p>这个程序看起来是运行正常的，可是真的是没有问题的吗？</p>

<h3></h3>

<p>我们简单的修改一下TimerCallback函数，强制调用一下GC，如下所示：</p>

```c#
    private static void TimerCallback(Object o)
    {
        // Display the date/time when this method got called.
        Console.WriteLine("In TimerCallback: " + DateTime.Now);
        // Force a garbage collection to occur.
        GC.Collect();
    }
```

<p>我们再次运行一下这个函数，得到了如下的输出：</p>

<p><a href="http://images.cnitblog.com/blog/163228/201306/20111612-7ffe48618fd14aca9a6f57ecd4548bfa.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://images.cnitblog.com/blog/163228/201306/20111612-b980e353236b4d92a0b062819ae6c284.png" width="414" height="137" /></a></p>

<p>我们可以看到，这次timer只被调用了一次！！！</p>

<h3></h3>





<p>这个时候大家应该能猜到原因了，我们的timer被垃圾回收了！！！</p>

<p>C#的垃圾回收采用了reference tracking的算法，大概的意思是说在执行垃圾回收时，所有的对象都默认认为是可以被回收的，然后遍历所有的roots（指向reference type的对象，包括类成员变量，静态变量，函数参数，函数局部变量），把这个root指向的对象标记成不能被回收的。</p>

<p>回到我们的代码，当我们强制调用GC.Collect()时，这个时候我们的timer t已经是一个没有被指向的对象了，于是垃圾回收就把t给回收了。这和C++的对象析构不太一样，C++的对象需要在出了作用域之后析构函数才会被调用到。</p>

<p>所以，即使我们没有显示的在这里调用GC.Collect()，但是我们不能确定什么时候CLR会调用GC，那个时候timer也就被回收了，总之，不能实现我们的意图。</p>

<h3>&#160;</h3>

<p>再来个有意思的，如果我们把上面的程序改成debug模式再运行，发现我们的timer还是能够正常工作的，就是说还是能看到每隔一秒就输出一条记录。这又是为什么呢？</p>

<p>因为Visual Studio为了让debug更方便在debug模式下编译时延长了局部变量的生命周期。比如说，假设你在一个局部变量最后一次被使用之后打了断点，但是这个时候你在watch窗口已经看不到那个局部变量的值了（被垃圾回收了），那是不是很抓狂。所以debug的编译器就做了这个小手脚。</p>

<h3>&#160;</h3>

<p>那如果要实现我们的初衷，就需要在Console.ReadLine之后还能保持一个对timer的引用，所以我们写了如下的代码：</p>


```c#
    public static void Main()
    {
        // Create a Timer object that knows to call our TimerCallback
        // method once every 1000 milliseconds.
        Timer t = new Timer(TimerCallback, null, 0, 1000);
        // Wait for the user to hit <Enter>
        Console.ReadLine();

        t = null;
    }
```



<p>很不幸，这在release下还是不行，因为编译器认为把一个对象置为null是没必要的，帮我们优化掉了！</p>

<p>所以正确的做法是：</p>

```c#
    public static void Main()
    {
        // Create a Timer object that knows to call our TimerCallback
        // method once every 1000 milliseconds.
        Timer t = new Timer(TimerCallback, null, 0, 1000);
        // Wait for the user to hit <Enter>
        Console.ReadLine();

        // This assignment will be removed by compiler optimization
        //t = null;

        // This will survive the GC
        t.Dispose();
    }
```

<p>这样就可以了。当然，我们也可以直接用using语句：</p>

```c#
    public static void Main()
    {
        // Create a Timer object that knows to call our TimerCallback
        // method once every 1000 milliseconds.
        using (new Timer(TimerCallback, null, 0, 1000))
        {
            // Wait for the user to hit <Enter>
            Console.ReadLine();
        }
    }
```


嗯，现在我们有了一个不会被垃圾回收掉的timer。希望对大家理解C#的垃圾回收有些帮助。
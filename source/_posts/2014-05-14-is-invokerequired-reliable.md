---
layout: post
title: "C#中只使用Invokerequired来判断是不是UI线程可靠吗？"
date: 2014-05-14 19:16
comments: true
tags: Programming
keywords:
description:
---

今天遇到一个C#的Crash，用windbg打开dump，加载sos之后一看，在4号线程出了一个`System.InvalidOperationException`，在这个地址上调用`!pe`。可以看到如下的异常信息：

```
Exception object:
Exception type: System.InvalidOperationException
Message: The calling thread cannot access this object because a different thread owns it.
InnerException: <none>
StackTrace (generated):
```

这个异常很常见了，就是非UI线程操作UI控件导致的。我们知道在C#中只有UI线程才能操作UI控件，如果一个工作线程操作UI控件，就会抛这个异常。通常的解决办法很简单，就是使用`Invokerequired`。如果返回True，说明不在UI线程，需要调用`Invoke`或者`BeginInvoke`来扔给UI线程操作。

可是打开call stack对应的代码，已经检查过`Invokerequired`了！！！这是怎么回事，是说`Invokerequired`不靠谱吗？

于是又回到MSDN的[Invokerequired](http://msdn.microsoft.com/en-us/library/system.windows.forms.control.invokerequired%28v=vs.110%29.aspx)，找到了如下的说明：
> This means that InvokeRequired can return false if Invoke is not required (the call occurs on the same thread), **or if the control was created on a different thread but the control's handle has not yet been created**.

> You can protect against this case by also checking the value of IsHandleCreated when InvokeRequired returns false on a background thread.

这就清楚了，如果Handle还没有创建好，那么即使在后台工作线程，`Invokerequired`也返回False。用如下的一个小程序来测试一下，代码很简单，就是在一个Form的构造函数里创建一个Timer，然后在这个Timer的Callback里面打印`Invokerequired`和`IsHandleCreated`的值。请注意这里一定要用`System.Threading.Timer`，因为Form里的那个Timer是运行在UI线程里的，不能用，具体参见我的博客[C#中5种timer的比较](/2013/07/09/compare-of-5-timers-in-csharp/)。另外关于这个`System.Threading.Timer`有一个坑，参见博客[谁动了我的timer？C#的垃圾回收和调试](/2013/06/20/where-is-my-timer-csharp-gc/)。关于`System.Timers.Timer`，也有一个坑，参见我的博客[.NET 2.0的Timer elapsed event 会自动catch住所有的exception](/2011/06/23/dot-net-2-timer-elapsed-event-will-catch-all-exception-for-you/)。

```csharp

using Timer = System.Threading.Timer;

namespace Form1
{
    public partial class Form1 : Form
    {
        private Timer m_Timer;
        public Form1()
        {
            m_Timer = new Timer(timer_callback, null, 0, 1);
            InitializeComponent();
        }

        private void timer_callback(object state)
        {
            Debug.WriteLine("IsHandleCreated: {0}", IsHandleCreated);
            Debug.WriteLine("InvokeRequired: {0}", InvokeRequired);
            Debug.WriteLine("Thread ID: {0}", Thread.CurrentThread.ManagedThreadId);
        }
    }
}
```

输出如下：（这个输出比较随机）

```bash
IsHandleCreated: False
InvokeRequired: False
Thread ID: 5
```

所以正确的写法是：

```csharp
if(control.IsDisposed || !control.IsHandleCreated)
{
    // UI is not ready, do something special here!!!
    return;
}

if(control.InvokeRequired)
{
    control.Invoke(action);
}
else
{
    action();
}
```

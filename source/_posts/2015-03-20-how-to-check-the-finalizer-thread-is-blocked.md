title: 如何判断C#的Finalizer线程有没有被阻塞
date: 2015-03-20 20:33:58
categories:
tags: [CSharp, Debug]
description: 本文介绍了如果判断C#的Finalizer线程有没有被阻塞。给出了一个由于Finalizer线程阻塞导致的句柄（Handle）数目超出Windows限制导致的崩溃问题，还附上了一小段代码用来查看当前进程Handle数目。
---

这两天在清理我的firefox打开的800多个页面，其中有很多已经是很早的，翻到了这个网页[Blocked Finalizer Thread](http://dotnetdebug.net/2005/06/22/blocked-finalizer-thread/)。依稀让我想起了以前遇到的那个由于GDI导致的OutOfMemory的crash。

我们知道Windows上每个进程最多能创建1万(10000)个Handle，参见[这里](http://blogs.msdn.com/b/oldnewthing/archive/2007/07/18/3926581.aspx)。所以如果你的进程创建的GDI对象达到1万的限制时就会抛出OutOfMemory的异常。我遇到的那个问题是一个Tree的每个节点上都会设置字体，但是每次都是new一个，然后会有一些情况导致这个树频繁刷新，就导致生成了很多的Font，但是为啥他们没有被垃圾回收呢？就和我上面列到的那篇文章有点关系了。

我们知道C#会用一个单独的Finalize线程来执行Finalize方法（根据程序不同的GC模型，也可能会有多个Finalize线程）。如果有什么问题导致这个线程被阻塞的话，就会导致积压了很多等待调用Finalize的对象，在我遇到的那个例子里，就是因为Finalize线程被阻塞了，导致Font的个数超过了进程限制。

什么原因会导致Finalize线程阻塞呢？很简单，就是Finalize方法被阻塞了。比如如果某个对象的Finalize方法中使用了同步对象，可以是Critical Section，Mutex，Semaphore等，但是这个对象被别的线程占用着，那么就会阻塞整个Finalize线程。另外一个中常见的问题是COM，一个STA的COM对象比如在STA线程进行释放，在C#中就是说这个COM对象必须在STA线程进行释放，如果STA线程在忙别的事情，就有可能导致Finalize线程被阻塞，我遇到的那个问题就是这个原因导致的。

那么怎么来检查Finalize线程有没有被阻塞呢？打开Windbg，可以attach到进程或者打开dump，然后加载SOS扩展，运行命令`!FinalizeQueue`，就可以列出所有等在Finalize Queue里的对象，如果在程序的多个时间抓取dump，看到这个数目在逐渐增多，就可以怀疑我们的Finalize线程被阻塞了。

如果想要确认的话，就需要切换到Finalize线程，运行`!threads`命令，然后在列出的线程中找到以`(Finalizer)`结尾的线程，然后切换过去，运行`k`命令列出call stack，如果看到例如`ZwWaitForSingleObject`或者`ZwWaitForMultipleObjects`的，基本就是在等待别的东西了，这个时候就可以去找找在等什么东西。另外也别忘了用`!clrstack`看看C#的call stack，有可能使用的是`Monitor.Enter`。如果在call stack中看到了`GetToSTA`，那就说明是上面提到的COM对象的情况。

对于这个问题怎么解决，在我的那个例子里，就共享了几个字体，没有每次都创建。对于其他的问题，首先一个是能不用Finalize方法就不用Finalize方法，如果是COM的话，可以显式的调用`Marshal.ReleaseComObject`来释放COM对象，这样就不需要劳驾Finalize线程来释放了。

下面再附一小段代码，它可以查看当前进程的Handle数目。

```c#
using System;
using System.Runtime.InteropServices;

namespace StreamWrite.Proceedings.Client
{
    public class HWndCounter
    {
        [DllImport("kernel32.dll")]
        private static extern IntPtr GetCurrentProcess();

        [DllImport("user32.dll")]
        private static extern uint GetGuiResources(IntPtr hProcess, uint uiFlags);

        private enum ResourceType
        {
            Gdi = 0,
            User = 1
        }

        public static int GetWindowHandlesForCurrentProcess(IntPtr hWnd)
        {
            IntPtr processHandle = GetCurrentProcess();
            uint gdiObjects = GetGuiResources(processHandle, (uint)ResourceType.Gdi);
            uint userObjects = GetGuiResources(processHandle, (uint)ResourceType.User);

            return Convert.ToInt32(gdiObjects + userObjects);
        }
    }
}
```

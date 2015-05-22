title: 避免在C#中使用析构函数Finalizer
date: 2015-05-22 21:14:04
categories:
tags: CSharp
description: C#的析构函数Finalizer坑太多，就不要用了。
---
Eric Lippert在When everything you know is wrong [part one](http://ericlippert.com/2015/05/18/when-everything-you-know-is-wrong-part-one/)和[part two](http://ericlippert.com/2015/05/21/when-everything-you-know-is-wrong-part-two/)中列举了一堆关于Finalizer的**错误**认识。

1. 把一个变量赋值为`null`会调用它之前指向的对象的Finalizer。  
1. 调用一个对象的`Dispose()`函数会调用它的Finalizer。  
1. 调用`GC.Collect()`会调用Finalizer。  
1. 只要一个对象有Finalizer，Finalizer就一定会被调到。  
1. 只要一个对象有Finalizer，并且没有调过`SuppressFinalize`，Finalizer就一定会被调到。  
1. 没有被任何对象引用的对象的Finalizer就一定会被调到。  
1. 如果一个对象被GC认为没有被任何对象引用，它的Finalizer就一定会被调到。  
1. 如果对象已经被放到了finalized queue，并且finalize线程已经被调度了，那它的Finalizer就一定会被调到。  
1. 如果对象已经被放到了finalized queue，并且finalize线程已经被调度了，并且finalizer都运行的很快，而且没有抛出异常，那它的Finalizer就一定会被调到。  
1. 局部变量只有在出了作用域它的Finalizer才可能会被调到。关于这个，可以参见我的博文[谁动了我的timer？C#的垃圾回收和调试](/2013/06/20/where-is-my-timer-csharp-gc/)  
1. Finalizer不可能被调用多次。看看这个API：[GC.ReRegisterForFinalize](https://msdn.microsoft.com/en-us/library/system.gc.reregisterforfinalize%28v=vs.110%29.aspx)。  
1. 被调到Finalizer的对象是死对象。  
1. 被调到Finalizer的对象是不能被别的对象在指到的。
1. 运行Finalizer的线程就是创建这个对象的线程。通常都在Finalize线程释放，但是COM的STA对象必须在STA线程释放，这个可以参见我的博文[如何判断C#的Finalizer线程有没有被阻塞](/2015/03/20/how-to-check-the-finalizer-thread-is-blocked/)。    
1. 运行Finalizer的线程就是GC线程。  
1. GC判断一个对象是死对象时就会调用它的Finalizer。  
1. Finalizer从来不会死锁。  
1. Fianlizer的运行顺序是可以预计的。  
1. 一个被调到Finalizer的对象可以安全访问其它对象。 
1. 运行Finalizer会释放内存。  
1. 被调到Finalizer的对象肯定是完全创建好的对象。  

再次强调一下，上面列举的这些都是**错误**的。Finalizer的坑这么多，那就还是不要用了。
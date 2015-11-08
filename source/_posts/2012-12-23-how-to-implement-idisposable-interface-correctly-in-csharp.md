---
layout: post
title: "C#中如何正确的实现IDisposable接口"
date: 2012-12-23
comments: true
tags: CSharp
---
<p>Stackoverflow上的这个<a href="http://stackoverflow.com/a/538238/304115">回答</a>是我见过的讲的最清楚的怎么正确实现<span style="color: #000000;">IDisposable</span>接口，我简单挑重点翻译翻译吧。：）</p>
<h3>&nbsp;</h3>
<p>Disposed的出现就是要解决一个问题，那就是释放非托管的资源。.NET的垃圾回收不知道怎么去释放非托管的资源。</p>
<p>所以，如果你的对象中有非托管的资源，你就需要提供一个函数给外面的人来释放它。我们有一个标准的名字：</p>

```csharp
public void Dispose()
```

<p>在C#中有个接口，只包含了这个函数，所以如果你的类需要释放非托管资源，就需要实现这个接口，实现了这个借口，意味着你承诺在Dispose方法中释放非托管资源。</p>

```
public interface IDisposable
{
    void Dispose();
}
```

<p>下面是个释放非托管资源的例子。</p>

```
public void Dispose()
{
   Win32.DestroyHandle(this.gdiCursorBitmapStreamFileHandle);
}
```
<p>这其实就搞定了：）但是你可以做的更好。</p>
<h3>&nbsp;</h3>
<p>如果你的对象中有托管的资源，而且很大，比如有个250m的bitmap。当然C#的垃圾回收会把它释放掉，但是更好的是我们能在不需要的时候就把它释放掉而不必等到垃圾回收。怎么做呢？我们已经有了一个方法来释放非托管的资源，好办，我们把在Dispose中把托管的资源也释放掉就好了，这样我们的Dispose干两件事情：</p>
<ol>
<li>释放非托管资源，因为必须</li>
<li>释放托管资源，因为这样好：）</li>
</ol>
<p>新的Dispose函数长这个样子：</p>

```
public void Dispose()
{
   //Free unmanaged resources
   Win32.DestroyHandle(this.gdiCursorBitmapStreamFileHandle);

   //Free managed resources too
   if (this.databaseConnection != null)
   {
      this.databaseConnection.Dispose();
      this.databaseConnection = null;
   }
   if (this.frameBufferImage != null)
   {
      this.frameBufferImage.Dispose();
      this.frameBufferImage = null;
   }
}
```
<p>看起来搞定了，但是，我们可以做的更好！</p>
<h3>&nbsp;</h3>
<p>如果我们的使用者没有调用Dispose怎么办呢？我们就会有资源泄露了！（当然只有非托管的资源泄露，因为托管的C#的垃圾回收会帮我们搞定它）</p>
<p>嗯，我们把这个Dispose放到Finalize里面，这样我的对象被销毁的时候，它就能自动调到，不错，代码如下：</p>

```
~MyObject()
{
    //we're being finalized (i.e. destroyed), call Dispose in case the user forgot to
    Dispose(); //<--Warning: subtle bug! Keep reading!
}
```
<p>但是，我们可能会引入一个bug。因为C#地垃圾回收是在后台运行，我们不知道它会先回收哪个对象，所以我们的对象中包含的托管资源可能已经被回收掉了，就会发生：</p>

```
public void Dispose()
{
   //Free unmanaged resources
   Win32.DestroyHandle(this.gdiCursorBitmapStreamFileHandle);

   //Free managed resources too
   if (this.databaseConnection != null)
   {
      this.databaseConnection.Dispose(); <-- crash, GC already destroyed it
      this.databaseConnection = null;
   }
   if (this.frameBufferImage != null)
   {
      this.frameBufferImage.Dispose(); <-- crash, GC already destroyed it
      this.frameBufferImage = null;
   }
}
```
<p>所以我们需要一种办法让在Finalize中调到的Dispose函数不去释放托管的资源，留给垃圾回收来做，我们习惯用这样的函数来实现：</p>
```
protected void Dispose(Boolean disposing)
```
<p>这个参数名字disposing有点不太好，我们改个名字吧：） </p>

```
protected void Dispose(Boolean freeManagedObjectsAlso)
{
   //Free unmanaged resources
   Win32.DestroyHandle(this.gdiCursorBitmapStreamFileHandle);

   //Free managed resources too, but only if i'm being called from Dispose
   //(If i'm being called from Finalize then the objects might not exist
   //anymore
   if (freeManagedObjectsAlso)  
   {    
      if (this.databaseConnection != null)
      {
         this.databaseConnection.Dispose();
         this.databaseConnection = null;
      }
      if (this.frameBufferImage != null)
      {
         this.frameBufferImage.Dispose();
         this.frameBufferImage = null;
      }
   }
}
```
<p>所以我们先前写的Dispose函数变成了这样：</p>

```
public void Dispose()
{
   Dispose(true); //i am calling you from Dispose, it's safe
}

public ~MyObject()
{
   Dispose(false); //i am *not* calling you from Dispose, it's *not* safe
}
```
<p>提醒一下，如果我们的类的父类也是IDisposable的，我们需要调用父类的Dispose函数。</p>

```
public Dispose()
{
   try
   {
      Dispose(true); //true: safe to free managed resources
   }
   finally
   {
      base.Dispose();
   }
}
```

<p>看起来搞定了，但是我们可以做的更好！</p>
<h3>&nbsp;</h3>
<p>如果我们的客户代码调用了Dispose，那意味着托管和非托管的资源都被释放掉了，但是随后C#的垃圾回收还会再次调用Dispose。不仅仅是浪费，而且如果万一我们没有在Dispose非托管资源时把它设成null，就是说它还指着东西，那就会被再释放一次，这可不是好玩的。</p>


```
protected void Dispose(Boolean iAmBeingCalledFromDisposeAndNotFinalize)
{
   //Free unmanaged resources
   Win32.DestroyHandle(this.gdiCursorBitmapStreamFileHandle); <--double destroy 
   //...
}
```
<p>修改这个问题的方法就是，如果已经调用过Dispose了，就不要再调Finalize了。</p>

```
public void Dispose()
{
   Dispose(true); //i am calling you from Dispose, it's safe
   GC.SuppressFinalize(this); //Hey, GC: don't bother calling finalize later
}
```
<p>好了，万事大吉：）</p>
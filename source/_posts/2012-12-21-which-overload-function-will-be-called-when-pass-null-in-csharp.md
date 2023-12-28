---
layout: post
title: "C#中把null作为参数传过去会调用哪个overload？"
date: 2012-12-21
comments: true
tags: Programming
---
<p><a href="http://stackoverflow.com/questions/13877501/why-do-i-get-an-exception-when-passing-null-constant-but-not-when-passing-a-n?newsletter=1&amp;nlcode=55866%7cc739">Stack Overflow</a>上有个有趣的问题，如果向下面这么写，没问题。</p>

```csharp
Console.WriteLine( String.Format( "{0}", (object)null) );
```
<p><br /><code></code></p>
<p><code>但是如果这么写,会出一个<code>ArgumentNullException。</code></code></p>

```
Console.WriteLine( String.Format( "{0}", null) );
```
<p><br />原因在于C#编译器会把null转换成最容易转到的类型，因为Fromat函数有下面几个重载：</p>

```
Format(String, Object)
Format(String, Object[])
Format(IFormatProvider, String, Object[])
Format(String, Object, Object)
Format(String, Object, Object, Object)
```
<p>而object[]可以转化成object，但是object不能转换成object[]，所以编译器会调用<a href="http://msdn.microsoft.com/en-us/library/b1csw23d.aspx">object[]</a>。</p>
<p>但是从MSDN离我们可以看出如果调用<a href="http://msdn.microsoft.com/en-us/library/b1csw23d.aspx">object[]</a>则需要保证format和ojbect[]都不是null，但是如果调用<a href="http://msdn.microsoft.com/en-us/library/fht0f5be.aspx">object</a>版本的，只要format不是null就行了。</p>
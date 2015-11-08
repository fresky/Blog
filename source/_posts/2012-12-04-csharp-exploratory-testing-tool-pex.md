---
layout: post
title: ".NET自动exploratory测试工具Pex"
date: 2012-12-04
comments: true
tags: [CSharp, Testing]
---
<p><a href="http://research.microsoft.com/en-us/projects/pex/">Pex</a>还是很好用的，安装之后可以在visual studio2008和2010中直接用，也能用命令行。</p>  <p>我写了下面这个很简单的小程序：</p>  

```csharp
	[PexClass]
    public class Calculator
    {
       [PexMethod]
       public int divide(int a,int b)
       {
           return a/b;
       }
    }
```

<p>用pex.exe命令行跑过之后出了一个report。</p>

{% limg Pex.png %}

<p>并且针对这3种情况自动生成3个unit test，很不错。</p>

<p>就是为啥还不支持2012呢？</p>
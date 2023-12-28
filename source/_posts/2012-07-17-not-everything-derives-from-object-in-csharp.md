---
layout: post
title: "C#的cast问题"
date: 2012-07-17
comments: true
tags: Programming
---
这篇<a href="http://blogs.msdn.com/b/ericlippert/archive/2009/08/06/not-everything-derives-from-object.aspx">Not everything derives from object - Fabulous Adventures In Coding - Site Home - MSDN Blogs</a>文章讲了为什么在C#里面

```csharp
            var enumerable = from bool b in new int[] {1, 2} select b;
            foreach (var element in enumerable)
            {
                Console.WriteLine(element);
            }
```


这个可以编译通过，但是运行会抛exception。<br /><br /><blockquote></blockquote>
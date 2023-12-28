---
layout: post
title: "c#怎么取到一个dll中所有的类型"
date: 2012-07-24
comments: true
tags: Programming
---
这篇<a href="http://haacked.com/archive/2012/07/23/get-all-types-in-an-assembly.aspx?utm_source=feedburner&amp;utm_medium=feed&amp;utm_campaign=Feed%3A+haacked+%28you%27ve+been+HAACKED%29">Get All Types in an Assembly</a>文章讲了怎么取到assembly中所有的types。<br />代码如下：<br />

```csharp
public static IEnumerable<Type> GetLoadableTypes(this Assembly assembly)
{
    if (assembly == null) throw new ArgumentNullException("assembly");
    try
    {
        return assembly.GetTypes();
    }
    catch (ReflectionTypeLoadException e)
    {
        return e.Types.Where(t => t != null);
    }
}
```
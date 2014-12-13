---
layout: post
title: "在C#中用Linq从属性文件中读取键值对Key-Value Pair"
date: 2014-03-28 08:41
comments: true
tags: CSharp
keywords: Linq, Key-Value Pari, Select, Where, GroupBy, ToDictionary
description: 在C#中用Linq从属性文件中读取键值对Key-Value Pair。使用Select, Where, GroupBy, ToDictionary。
---

我们经常需要读取一些如下的属性文件，然后把他们放到一个Dictionary里面。
```
Name = Dawei XU
Email = dawei.xu@gmail.com
```

在C#中可以用Linq很方便的实现这个功能。先看代码：

```c#
File.ReadAllLines(fileName)
.Select(line => line.Split(new[] {'='}, 2, StringSplitOptions.RemoveEmptyEntries))
.Where(split => split.Length == 2)
.ToDictionary(split => split[0].Trim(), split => split[1].Trim());
```
用到了如下Linq扩展方法：

- [Enumerable.Select](http://msdn.microsoft.com/en-us/library/system.linq.enumerable.select.aspx)，用来对每一行做split操作，分隔出Key和Value。注意这里要写上分隔成两段。另外要`StringSplitOptions.RemoveEmptyEntries`。
- [Enumerable.Where](http://msdn.microsoft.com/en-us/library/system.linq.enumerable.where%28v=vs.110%29.aspx)，用来过滤到不符合Key-Value的行。
- [Enumerable.ToDictionary](http://msdn.microsoft.com/en-us/library/system.linq.enumerable.todictionary%28v=vs.110%29.aspx)，把分隔成两段的第一段作为Key，第二个作为Value存在Dictionary里面。


但是这个做法有一个潜在的问题，就是如果属性文件中有重复的Key出现，比如：
```
Email = dawei.xu@hotmail.com
Name = Dawei XU
Email = dawei.xu@gmail.com
```

就会抛出[ArgumentException](http://msdn.microsoft.com/en-us/library/system.argumentexception.aspx)

> An element with the same key already exists in the Dictionary<TKey, TValue>.

解决办法就是用[Enumerable.GroupBy](http://msdn.microsoft.com/en-us/library/system.linq.enumerable.groupby%28v=vs.110%29.aspx)把重复的Key合并在一起，然后可以根据需要使用第一次出现的或者最后一次出现的，下面的例子是采用最后一次出现的。同时因为用了`GroupBy`，最后的`ToDictionary`需要做一些相应的修改。

```c#
File.ReadAllLines(fileName)
.Select(line => line.Split(new[] {'='}, 2, StringSplitOptions.RemoveEmptyEntries))
.Where(split => split.Length == 2)
.GroupBy(split=>split[0].Trim())
.ToDictionary(group => group.Key.Trim(), group => group.Last().Last().Trim());
```
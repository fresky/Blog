title: C#中如何操作2个list
date: 2015-03-30 17:24:36
categories:
tags: CSharp
description: 本文介绍了在C#中如何用Enumerable.Select和Enumerable.Zip来简化操作2个list。
---

假设我们有两个list，一个是{1,2,3}，一个是{4,5,6}，如果我们想让每个元素分别相乘，然后生成一个新的list{4,10,18}，如何写呢？

最直观的做法：

```c#
List<int> a = new List<int>(){1,2,3};
List<int> b = new List<int>(){4,5,6};
List<int> c = new List<int>();

for (int i = 0; i < a.Count; i++)
{
	c.Add(a[i]*b[i]);
}
```

如果到了.NET 3.5的Linq，那么我们可以用[Enumerable.Select](https://msdn.microsoft.com/en-us/library/vstudio/system.linq.enumerable.select%28v=vs.110%29.aspx)。
```c#
List<int> a = new List<int>(){1,2,3};
List<int> b = new List<int>(){4,5,6};

var c = a.Select((x, i) => b[i] * x);
```

如果到了.NET 4，我们可以用[Enumerable.Zip](https://msdn.microsoft.com/en-us/library/vstudio/dd267698%28v=vs.110%29.aspx)

```c#
List<int> a = new List<int>(){1,2,3};
List<int> b = new List<int>(){4,5,6};

var c = a.Zip(b, (x, y) => x*y);
```
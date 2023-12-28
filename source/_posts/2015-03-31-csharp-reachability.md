title: C#编译器怎么检查代码是否会执行
date: 2015-03-31 22:48:33
categories:
tags: Programming
description: 本文介绍了C#编译器是怎么检查代码是否可达的，C# Reachability。
---
今天翻了一篇Eric Lippert的旧文章[C# Reachability](http://blog.coverity.com/2013/11/06/c-reachability/)，里面详细解释了C#编译器是怎么判断代码是否可达（Unreachable code）的。

文章开始于一个问题，为什么下面的代码在编译时只有`Condition()`不可达的警告，但那时没有`Alternative()`不可达的警告。

```csharp
if (true || Condition())
  Consequence();
else
  Alternative();
```

原因在于C#编译器对于可达性的检查大概是这样子的。

1. 基于编译时常量检查的结果做可达性检查，如果不可达就给出警告。

常量检查的前提是**每个部分都是常量**。这就是上面的例子为什么`Condition()`有不可达的警告。 

2. 优化（如果用户开优化了），包含算数优化，比如`(123 != 0 * someIntegerLocalVariable)`会被优化成true。

3. 基于上面优化的结果在做一个可达性检查，对于不可达的代码不会产生编译结果，但是这时不会有不可达的警告。

至于为什么上面第三条不给出不可达的警告主要原因在于，如果用户没有开优化，那么这个不可达检查不会发生。如果这个也给出警告的话，就会导致用户开没开优化得到的警告不一致。

另外，C#编译器会放松对第一次不可达检查时找出的不可达代码的检查，比如下面的代码不会报错。

```csharp
static void Main(string[] args)
{
	int a;
	if (false)
	{
		int b = m(a);
		Console.WriteLine(b);
	}
	
	Console.WriteLine();
}

private static int m(int i)
{
	return i + 1;
}
```

但是如果改成下面这个样子，就会报错了，**Use of unassigned local variable 'a'**。


```csharp
static void Main(string[] args)
{
	bool condition = true;
	int a;
	if (false&&condition)
	{
		int b = m(a);
		Console.WriteLine(b);
	}
	
	Console.WriteLine();
}

private static int m(int i)
{
	return i + 1;
}
```
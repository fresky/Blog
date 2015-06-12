title: Moq4在.NET3.5和.NET4版本之间的差异
date: 2015-06-12 11:14:21
categories:
tags: [CSharp, Tool]
description: Moq4为了实现返回值基于参数的功能支持超过4个参数的方法，导致Moq做了2个dll，分别是基于.NET3.5和.NET4。就是我们下载Moq时看到的Moq35和Moq40两个文件夹。这两个dll在大部分情况下可以混用，但是如果测试方法中包含要Mock超过4个参数的方法，并且需要根据参数值来决定返回值时，混用Moq35和Moq40会导致测试失败。
---
[Moq](https://github.com/Moq/moq4)是应用最广泛的一个C#的Mock框架。它有一个功能是在Mock一个方法时能根据这个方法的输入参数来设置返回值。如下所示：
```c#
mock.Setup(x => x.DoSomething(It.IsAny<string>())).Returns((string s) => s.ToLower());
```

但是在Moq的版本4之前只支持最多4个参数，在版本4中开始支持最多16个参数。它的实现是通过`System.Func`来实现的，但是因为.NET4之前没有`System.Func`，所以Moq4的.NET35版本和.NET40版本会有所差异。

假设我们需要Mock多于4个参数的方法返回值时，比如如下代码：
```c#
mock.Setup(
x =>
x.DoSomething(It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>()).Returns((string p1, string p2, string p3, string p4, string p5) => p1+p2+p3+p4+p5);
```

如果我们的测试项目的.NET版本是4以上，但是在编译时引用的是Moq35，那么在编译时会遇到一个如下的编译警告：
```
warning CS1685: The predefined type 'System.Func' is defined in multiple assemblies in the global alias;
```

这个原因是因为Moq35的版本中自带了一个`System.Func`的定义，如下图所示。但是这个在.NET4以上的版本中已经定义过了。

{% limg moq35.jpg %}

我们忽略这个警告，在运行测试时如果用的是Moq35，那么没有问题，可以通过。但是如果运行时使用的是Moq40，就会遇到这样的错误。这是因为在Moq40的dll里并没有包含`System.Func`的定义。

```
Test method *** threw exception: 
System.TypeLoadException: Could not load type 'System.Func`10' from assembly 'Moq, Version=4.0.10827.0, Culture=neutral, PublicKeyToken=69f491c39445e920'.
```

如果我们在编译时引用的是Moq40，但是在运行时使用的是Moq35，那么会遇到如下的错误：
```
Test method *** threw exception: System.MissingMethodException: Method not found: 
'Moq.Language.Flow.IReturnsResult`1<!0> Moq.Language.IReturns`2.Returns(System.Func`10<!!0,!!1,!!2,!!3,!!4,!!5,!!6,!!7,!!8,!1>)'.
```

如果想无论编译用的什么版本，运行时是什么版本，测试都能跑过的话，需要手动Mock超过4个参数的函数，比如如下：
```
internal class Mock : Real
{
	public override string DoSomething(string p1, string p2, string p3, string p4, string p5)
	{
		return p1+p2+p3+p4+p5;
	}
}
```

当然了，这么做仅仅需要在一个因为各种原因混用了Moq35和Moq40的情况下才需要。更好的办法是只用一个Moq版本。

**总结一下：**

Moq4为了实现返回值基于参数的功能支持超过4个参数的方法，导致Moq做了2个dll，分别是基于.NET3.5和.NET4。就是我们下载Moq时看到的Moq35和Moq40两个文件夹。这两个dll在大部分情况下可以混用，但是如果测试方法中包含要Mock超过4个参数的方法，并且需要根据参数值来决定返回值时，混用Moq35和Moq40会导致测试失败。
title: 要注意null合并运算符的优先级比+还要低
date: 2015-03-23 22:18:41
categories:
tags: Programming
description: C#的null合并运算符的优先级比加号还要低，注意在和别的运算符用在一起时要记得加括号。
---

[null合并运算符](https://msdn.microsoft.com/en-us/library/ms173224.aspx)是一个非常有用的运算符，有了它可以让代码非常简洁，省去了很多繁琐的空检查，但是有一点要注意的是它的优先级比加号还要低，可以参见Eric Lippert写的这篇[Null Coalescing Bugs in C#](http://blog.coverity.com/2013/10/23/null-coalescing-bugs/#.VRAepOFqP7G)，所以在和别的运算符用在一起时要记得加括号。

假设有如下的代码：

```csharp
int? a = null;
int? b = null;
Console.WriteLine(a ?? 1 + b ?? 2);
```

他的输出结果是2，而不是3！原因就是因为`??`的优先级比`+`还要低，所以上面的语句相当于`a ??( (1 + b) ?? 2)`。就是说，首先看`a`是`null`，所以应该取`(1 + b) ?? 2`。而`(1 + b)`还是`null`，所以结果是2。

如果想让结果为3，需要这么写`(a ?? 1) + (b ?? 2)`。
			
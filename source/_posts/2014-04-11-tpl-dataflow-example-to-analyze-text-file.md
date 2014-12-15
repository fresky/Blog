---
layout: post
title: "一个使用C#的TPL Dataflow Library的例子：分析文本文件中词频"
date: 2014-04-11 16:21
comments: true
tags: [CSharp]
keywords: csharp, tpl, dataflow, text,文本处理，工作流，多线程
description: 使用Task.ContinueWith和TPL的Dataflow来解决工作流（管道）问题。在本文的例子中是分析文本文件中词频。
---
[TPL Dataflow Library](http://msdn.microsoft.com/en-us/library/hh228603%28v=vs.110%29.aspx)是微软提供的一个关于并发的类库。它主要想解决的就是工作流的问题。比如我们的目标任务需要有一系列的模块来一次完成，前一个模块的输出是后一个模块的输入，这样就构成了一个管道，利用TPL Dataflow Library可以很方便的实现这个功能。

我在之前的博客[使用Linux的命令行工具做简单的文本分析](/2013/06/18/how-to-analyze-text-file-with-linux-command-line-tools/)中介绍过如何用Linux的命令行工具找出一个文章中的词频分布，下面我们用TPL Dataflow和普通的Task和ContinueWith来分别实现，看看其中的区别。

问题描述
---
首先描述一下我们的问题，我们有一个文件夹，里面放了一些文本文件，我们需要找出每个文件中词频最高的前5个词。把这个问题分解成几个子模块，分别是： 

1. 从文件中读取字符串  
1. 创建文字列表  
1. 过滤文字列表统计词频  
1. 打印出现频率最高的5个词  

子问题解决
---

 1 . 从文件中读取字符串
```c#
private static string readFileText(string fileName)
{
	Console.WriteLine("Reading '{0}'...", fileName);
	return File.ReadAllText(fileName);
}
```
 2 . 创建文字列表
```c#
private static string[] createWordList(string text)
{
	Console.WriteLine("Creating word list...");

	// Remove common punctuation by replacing all non-letter characters  
	// with a space character to. 
	char[] tokens = text.ToArray();
	for (int i = 0; i < tokens.Length; i++)
	{
		if (!char.IsLetter(tokens[i]))
			tokens[i] = ' ';
	}
	text = new string(tokens);

	// Separate the text into an array of words. 
	return text.Split(new char[] {' '},
		StringSplitOptions.RemoveEmptyEntries);
}
```
 3 . 过滤文字列表统计词频
```c#
private static Dictionary<string, int> fileterWordList(string[] words)
{
	Console.WriteLine("Group and ordering word list...");

	return words.Where(word => word.Length > 3).GroupBy(word => word)
		.OrderByDescending(group => group.Count())
		.ToDictionary(group => group.Key, group => group.Count());
}
```
 4 . 打印出现频率最高的5个词
```c#
private static void printWordFrequency(Dictionary<string, int> dic)
{
	Console.WriteLine("Top 5:");
	int top = 0;
	foreach (var pair in dic)
	{
		Console.WriteLine("{0}   :   {1}", pair.Key, pair.Value);
		if (++top > 5)
			break;
	}
}
```

使用Task实现多线程
---
单线程没啥好说的，直接连起来调用就行了，不多废话。

如果要多线程，可以直接用Task。使用`ContinueWith`。
```c#
private static void getWordFrequencyWithTask()
{
	List<Task> allFinalTasks = new List<Task>();
	foreach (var fileName in Directory.GetFiles(@"D:\Temp\test"))
	{
		allFinalTasks.Add(Task<string>.Factory.StartNew(() => readFileText(fileName))
			.ContinueWith<string[]>(t => createWordList(t.Result))
			.ContinueWith<Dictionary<string, int>>(t => fileterWordList(t.Result))
			.ContinueWith(t => printWordFrequency(t.Result)));

	}
	Task.WaitAll(allFinalTasks.ToArray());
}
```

使用TPL的Dataflow
---
首先要注意的是Dataflow需要用Nuget加进来。用到的类和函数如下：

1. `TransformBlock`：有输入有输出，一般用于非最后一个处理模块。  
1. `ActionBlock`：只有输入，没有输出，一般用于最后一个处理模块。  
1. `LinkTo`：把模块关联起来。  
1. `Completion.ContinueWith`：表明这个模块完成后标识下一个模块完成。  
1. `Complete()`：告诉第一个模块已经没有新的输入了。  

代码如下：
```c#
private static void getWordFrequencyWithTPLDataflow()
{
	// 1st, Read the files to strings
	var read = new TransformBlock<string, string>(fileName => readFileText(fileName));

	// 2nd, Separates the specified text into an array of words. 
	var create = new TransformBlock<string, string[]>(text => Program.createWordList(text));

	// 3rd, Removes short words, orders the resulting words by frequency. 
	var filter = new TransformBlock<string[], Dictionary<string, int>>(words => fileterWordList(words));

	// 4th, Prints the provided palindrome to the console.     
	var print = new ActionBlock<Dictionary<string, int>>(dic => printWordFrequency(dic));

	// 
	// Connect the dataflow blocks to form a pipeline. 
	//

	read.LinkTo(create);
	create.LinkTo(filter);
	filter.LinkTo(print);

	// 
	// For each completion task in the pipeline, create a continuation task 
	// that marks the next block in the pipeline as completed. 
	// A completed dataflow block processes any buffered elements, but does 
	// not accept new elements. 
	//

	read.Completion.ContinueWith(t =>
	{
		if (t.IsFaulted) ((IDataflowBlock) create).Fault(t.Exception);
		else create.Complete();
	});
	create.Completion.ContinueWith(t =>
	{
		if (t.IsFaulted) ((IDataflowBlock) filter).Fault(t.Exception);
		else filter.Complete();
	});
	filter.Completion.ContinueWith(t =>
	{
		if (t.IsFaulted) ((IDataflowBlock)print).Fault(t.Exception);
		else print.Complete();
		
	});


	foreach (var fileName in Directory.GetFiles(@"D:\Temp\test"))
	{
		read.Post(fileName);
	}
	read.Complete();
	print.Completion.Wait();
}
```

完整代码参见[Gist](https://gist.github.com/fresky/10632899)
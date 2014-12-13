---
layout: post
title: "使用CLRMD编写一个自己的C#调试器"
date: 2014-02-17 19:33
comments: true
tags: [CSharp, Debug]
keywords: CLRMD, debugger, csharp
description: 使用CLRMD编写一个自己的C#调试器
---

[CLR Memory Diagnostics (ClrMD)](https://www.nuget.org/packages/Microsoft.Diagnostics.Runtime)是微软提供的一个库，可以使用这个库的API来分析CLR的dump文件，或者直接attach到一个CLR的进程上。使用这些API可以实现和Windbg的SOS扩展类似的功能。

使用起来非常简单，可以通过nuget把这个reference加上，然后就是`DataTarget`，`ClrRuntime`，`ClrHeap`，`ClrType`这几个类，API都很清楚。下面是两个小的例子。

第一个是attach到一个进程上，然后把这个进程内的所有字符串打印出来，这样如果一个程序没有对密码加密，就能很容易的被看到的。

```c#
using (DataTarget dataTarget = DataTarget.AttachToProcess(pid, 5000))
{
	Console.WriteLine();
	string dacLocation = dataTarget.ClrVersions[0].TryGetDacLocation();
	ClrRuntime runtime = dataTarget.CreateRuntime(dacLocation);
	ClrHeap heap = runtime.GetHeap();
	foreach (ulong obj in heap.EnumerateObjects())
	{
		ClrType type = heap.GetObjectType(obj);

		if (typeof (string).ToString().Equals(type.Name))
		{
			Console.WriteLine(type.GetValue(obj));
		}
	}
}
```

第二个是打开一个dump，遍历所有线程，把call stack打印出来。
```c#
using (DataTarget dataTarget = DataTarget.LoadCrashDump(file))
{
	string dacLocation = dataTarget.ClrVersions[0].TryGetDacLocation();
	ClrRuntime runtime = dataTarget.CreateRuntime(dacLocation);
	foreach (ClrThread thread in runtime.Threads)
	{
		Console.WriteLine("ThreadID: {0:X}", thread.OSThreadId);
		Console.WriteLine("Callstack:");

		foreach (ClrStackFrame frame in thread.StackTrace)
			Console.WriteLine("{0,12:X} {1,12:X} {2}", frame.InstructionPointer, frame.StackPointer,
				frame.DisplayString);
		Console.WriteLine();
	}
}
```
---
layout: post
title: "使用CLRMD时通过Symbol Server找Dac的位置来初始化ClrRuntime"
date: 2014-02-18 14:38
comments: true
tags: [CSharp, Debug]
keywords: CLRMD, debugger, csharp, symbol server
description: 使用CLRMD时通过Symbol Server找Dac的位置来初始化ClrRuntime
---

在我昨天的博客[使用CLRMD编写一个自己的C#调试器](/2014/02/17/use-clrmd-to-create-your-own-csharp-debugger/)介绍了一下如果使用CLRMD的API，其中第二个例子是打开一个CLR的dump，来遍历所有线程，把call stack打印出来，类似在每个线程上运行`!clrstack`命令。

昨天的那个小例子有一个潜在的问题，就是如果dump是别的机器生成的，自己机器上没有装匹配的clr时，函数`TryGetDacLocation`会返回`null`。这样就不能正确的初始化`ClrRuntime`了。

[ClrMD: Fetching DAC Libraries From Symbol Servers](http://winterdom.com/2013/05/clrmd-fetching-dac-libraries-from-symbol-servers)介绍了如果通过使用`dbghelp.dll`（在安装windbg或者debugdia时会装在你的电脑上）来从symbol路径里面找Dac。我参考这篇文章写了一个读取dump，打印异常的小程序。

先看输出吧，和在windbg里面用sos扩展敲`!pe`的效果很类似，不过自动把inner exception也打印出来了。

```
ThreadID: 11EC
Managed ThreadID: 1
Exception Address: 244A086C
Exception Type: System.Reflection.TargetInvocationException
Message: Exception has been thrown by the target of an invocation.
Call stack:
           SP           IP Function
            0            0 System.RuntimeMethodHandle._InvokeMethodFast(System.Object, System.Object[], System.SignatureStruct ByRef, System.Reflection.MethodAttributes, System.RuntimeTypeHandle)
       33E458     6FD15637 System.RuntimeMethodHandle.InvokeMethodFast(System.Object, System.Object[], System.Signature, System.Reflection.MethodAttributes, System.RuntimeTypeHandle)
...
HResult: 80131604
Inner Exception:
Exception Type: System.OutOfMemoryException
Message: Out of memory.
Call stack:
           SP           IP Function
       33CD04     6F313341 System.Drawing.Graphics.FromHdcInternal(IntPtr)
       33CD14     6F2115F0 System.Drawing.Font.ToLogFont(System.Object)
       33CD44     6F211538 System.Drawing.Font.ToHfont()
       33CD74     6E7E37DE System.Windows.Forms.Control+FontHandleWrapper..ctor(System.Drawing.Font)
       33CD80     6EEAB7FA System.Windows.Forms.OwnerDrawPropertyBag.get_FontHandle()
       33CD90     6F04AEA3 System.Windows.Forms.TreeView.CustomDraw(System.Windows.Forms.Message ByRef)
       33CEA8     6E80C16C System.Windows.Forms.TreeView.WmNotify(System.Windows.Forms.Message ByRef)
       33CF28     6E80BDB6 System.Windows.Forms.TreeView.WndProc(System.Windows.Forms.Message ByRef)
...
HResult: 8007000E
Press any key to continue . . .
```

加载dump的代码：
```c#
private static void LoadDump(string file)
{
	using (DataTarget dataTarget = DataTarget.LoadCrashDump(file))
	{
		// use microsoft public symbol server to find the DAC
		string symcache = @"D:\NotBackedUp\SymCache";
		DacLocator dacloc = DacLocator.FromPublicSymbolServer(symcache);
		string dacLocation = dacloc.FindDac(dataTarget.ClrVersions[0]);
		
		ClrRuntime runtime = dataTarget.CreateRuntime(dacLocation);
		foreach (ClrThread thread in runtime.Threads)
		{
			if (thread.CurrentException != null)
			{
				
				Console.WriteLine("ThreadID: {0:X}", thread.OSThreadId);
				Console.WriteLine("Managed ThreadID: {0:X}", thread.ManagedThreadId);
				Console.WriteLine("Exception Address: {0:X}", thread.CurrentException.Address);

				ClrException ex = thread.CurrentException;
				PrintException(ex);
				if (ex.Inner !=null)
				{
					Console.WriteLine("Inner Exception:");
					PrintException(ex.Inner);
				}
			}
		}
	}
}
```

打印exception的代码：
```c#
private static void PrintException(ClrException ex)
{
	Console.WriteLine("Exception Type: {0}", ex.Type.Name);
	try
	{
		Console.WriteLine("Message: {0}", ex.Message);
	}
	catch
	{
		Console.WriteLine("Message: No Message due to mini dump!");
	}
	
	Console.WriteLine("Call stack:");
	Console.WriteLine(" {0,12:X} {1,12:X} {2}", "SP", "IP", "Function");
	foreach (var frame in ex.StackTrace)
	{
		Console.WriteLine(" {0,12:X} {1,12:X} {2}", frame.StackPointer, frame.InstructionPointer, frame.DisplayString);
	}
	try
	{
		Console.WriteLine("HResult: {0:X}", ex.HResult);
	}
	catch
	{
	}
}
```

注意这里有两个try，是因为我发现如果打开的mini dump，在拿Messages和HResult时会出exception。

通过symbol路径寻找Dac的代码如下：（主要是上面提到的那篇文章的代码）
```c#
public class DacLocator : IDisposable
{
	const int sfImage = 0;
	[DllImport("dbghelp.dll", SetLastError = true)]
	static extern bool SymInitialize(IntPtr hProcess, String symPath, bool fInvadeProcess);

	[DllImport("dbghelp.dll", SetLastError = true)]
	static extern bool SymCleanup(IntPtr hProcess);

	[DllImport("dbghelp.dll", SetLastError = true)]
	static extern bool SymFindFileInPath(IntPtr hProcess, String searchPath, String filename, uint id, uint two, uint three, uint flags, StringBuilder filePath, IntPtr callback, IntPtr context);

	[DllImport("kernel32.dll", SetLastError = true)]
	static extern LibrarySafeHandle LoadLibrary(String name);

	[DllImport("kernel32.dll", SetLastError = true)]
	static extern bool FreeLibrary(IntPtr hModule);


	private String searchPath;
	private LibrarySafeHandle dbghelpModule;
	private Process ourProcess;

	private DacLocator(String searchPath)
	{
		this.searchPath = searchPath;
		ourProcess = Process.GetCurrentProcess();
		dbghelpModule = LoadLibrary("dbghelp.dll");
		if (dbghelpModule.IsInvalid)
		{
			throw new Win32Exception(String.Format("Could not load dbghelp.dll: {0}", Marshal.GetLastWin32Error()));
		}
		if (!SymInitialize(ourProcess.Handle, searchPath, false))
		{
			throw new Win32Exception(String.Format("SymInitialize() failed: {0}", Marshal.GetLastWin32Error()));
		}
	}

	public static DacLocator FromPublicSymbolServer(String localCache)
	{
		return new DacLocator(String.Format("SRV*{0}*http://msdl.microsoft.com/download/symbols", localCache));
	}
	public static DacLocator FromEnvironment()
	{
		String ntSymbolPath = Environment.GetEnvironmentVariable("_NT_SYMBOL_PATH");
		return new DacLocator(ntSymbolPath);
	}
	public static DacLocator FromSearchPath(String searchPath)
	{
		return new DacLocator(searchPath);
	}

	public String FindDac(ClrInfo clrInfo)
	{
		String dac = clrInfo.TryGetDacLocation();
		if (String.IsNullOrEmpty(dac))
		{
			dac = FindDac(clrInfo.DacInfo.FileName, clrInfo.DacInfo.TimeStamp, clrInfo.DacInfo.FileSize);
		}
		return dac;
	}
	public String FindDac(String dacname, uint timestamp, uint fileSize)
	{
		// attempt using the symbol server
		StringBuilder symbolFile = new StringBuilder(2048);
		if (SymFindFileInPath(ourProcess.Handle, searchPath, dacname,
			timestamp, fileSize, 0, 0x02, symbolFile, IntPtr.Zero, IntPtr.Zero))
		{
			return symbolFile.ToString();
		}
		else
		{
			throw new Win32Exception(String.Format("SymFindFileInPath() failed: {0}", Marshal.GetLastWin32Error()));
		}
	}

	public void Dispose()
	{
		if (ourProcess != null)
		{
			SymCleanup(ourProcess.Handle);
			ourProcess = null;
		}
		if (dbghelpModule != null && !dbghelpModule.IsClosed)
		{
			dbghelpModule.Dispose();
			dbghelpModule = null;
		}
	}

	class LibrarySafeHandle : SafeHandleZeroOrMinusOneIsInvalid
	{
		public LibrarySafeHandle()
			: base(true)
		{
		}
		public LibrarySafeHandle(IntPtr handle)
			: base(true)
		{
			this.SetHandle(handle);
		}
		protected override bool ReleaseHandle()
		{
			return FreeLibrary(this.handle);
		}
	}
}
```
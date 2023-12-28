title: 怎么从代码中拿到栈回溯信息（call stack trace）
date: 2015-06-21 21:59:37
categories:
tags: [Programming, Debug]
description: 本文介绍如何在C++和C#代码中得到当前的栈回溯信息（call stack trace）。C#使用了System.Diagnostics.StackTrace，C++使用了CaptureStackBackTrace和DbgHelp中的SymSetOptions，SymInitialize，SymCleanup，SymFromAddr，SymGetLineFromAddr64。
---
在上一篇博客[调试内存泄漏问题的一些经验](/2015/06/21/how-to-attack-the-memory-leak-issue/)中提到过通过gflags设置“Create user mode stack trace database”可以存储生成对象的调用栈信息（call stack trace），这样以后可以很方便的看到每个泄漏的对象是怎么产生的。

假设我们不用gflags的这个设置，自己在代码中得到栈回溯信息，怎么做呢？

# C&#35;

C#实现这个功能很简单，直接使用[System.Diagnostics.StackTrace](https://msdn.microsoft.com/en-us/library/system.diagnostics.stacktrace%28v=vs.110%29.aspx)就行了，唯一要注意的是在创建`StackTrace`时传`true`作为参数。示例代码如下：

```csharp
var stackTrace = new StackTrace(true); // true means capturing source information
Console.WriteLine(stackTrace);
```

# C++
C++中获取栈回溯信息需要使用[CaptureStackBackTrace](https://msdn.microsoft.com/en-us/library/windows/desktop/bb204633%28v=vs.85%29.aspx)。  

如果只是在Visual Studio中看的话，Visual Studio 2013已经能自动把指令地址转换成函数名，源代码行数显示在Watch窗口中了。也可以[写Natvis来使它显示的更加友好](http://blogs.msdn.com/b/vcblog/archive/2014/01/23/examining-stack-traces-of-objects-using-visual-studio-2013.aspx)。关于Natvis的使用方法，可以参加我的博客[用Natvis定制C++对象在Visual Studio调试时如何显示](/2015/06/17/customize-your-debugger-for-cpp-object/)。

如果我们想自己得到函数名和源代码行数，相对来说就有点麻烦了，需要使用[DbgHelp](https://msdn.microsoft.com/en-us/library/windows/desktop/ms680578%28v=vs.85%29.aspx)中的下面几个API：
1. [SymSetOptions](https://msdn.microsoft.com/en-us/library/windows/desktop/ms681366%28v=vs.85%29.aspx)：设置Symbol选项，我们需要设置`SYMOPT_LOAD_LINES`来获取源文件行数信息。  
1. [SymInitialize](https://msdn.microsoft.com/en-us/library/windows/desktop/ms681351%28v=vs.85%29.aspx)：初始化Symbol，它的构造函数有3个参数。第一个是进程句柄。第二个是Symbol Path，如果这个参数为`NULL`，那会从当前工作目录，环境变量的`_NT_SYMBOL_PATH`和`_NT_ALTERNATE_SYMBOL_PATH`中加载Symbo。第三个参数是`fInvadeProcess`，如果这个参数是`TURE`会对所有module自动调用[SymLoadModule64](https://msdn.microsoft.com/en-us/library/windows/desktop/ms681352(v=vs.85).aspx)。  
1. [SymCleanup](https://msdn.microsoft.com/en-us/library/windows/desktop/ms680696%28v=vs.85%29.aspx)：释放所有资源。   
1. [SymFromAddr](https://msdn.microsoft.com/en-us/library/windows/desktop/ms681323%28v=vs.85%29.aspx)：针对给定地址查询Symbol信息。  
1. [SymGetLineFromAddr64](https://msdn.microsoft.com/en-us/library/windows/desktop/ms681330%28v=vs.85%29.aspx)：针对给定地址查询源代码行数。   
**注意DbgHelp的所有函数都是单线程的，所以最好在进程启动的时候调用`SymInitialize`，进程退出的时候调用`SymCleanup`。其它所有函数调用都要做好同步。** 

示例代码如下：
```c++
#pragma comment(lib, "dbghelp.lib")
#include <windows.h>
#include <DbgHelp.h>
#include <iostream>

class StackTrace {
private:
	static const int m_MaxFrameCount = 62;
	int m_FrameCount;
	void* m_Frames[m_MaxFrameCount];
public:
	StackTrace() {
		m_FrameCount = CaptureStackBackTrace(1, m_MaxFrameCount, m_Frames, NULL);
	}
	void Print()
	{
		HANDLE process = GetCurrentProcess();
		
		SymSetOptions(SYMOPT_LOAD_LINES); // set symbol option to load the source code lines
		SymInitialize(process, NULL, TRUE);  // init symbol
		
		char buffer[sizeof(SYMBOL_INFO) + MAX_SYM_NAME * sizeof(TCHAR)]; // define the PSYMBOL_INFO
		PSYMBOL_INFO symbol = (PSYMBOL_INFO)buffer;
		symbol->SizeOfStruct = sizeof(SYMBOL_INFO);
		symbol->MaxNameLen = MAX_SYM_NAME;

		IMAGEHLP_LINE64 line; // define the IMAGEHLP_LINE64
		line.SizeOfStruct = sizeof(IMAGEHLP_LINE64);

		for (int i = 0; i < m_FrameCount; i++)
		{
			DWORD64  dw64Displacement;
			SymFromAddr(process, (DWORD64)(m_Frames[i]), &dw64Displacement, symbol); // get symbol info
			std::cout << symbol->Name;
			DWORD dwDisplacement;
			if (SymGetLineFromAddr64(process, (DWORD64)(m_Frames[i]), &dwDisplacement, &line)) // get line info
			{
				std::cout << symbol->Name << ", " << line.FileName << ", line " << line.LineNumber;
			}
			std::cout << std::endl;
		}

		SymCleanup(process);
	}
};
```

会打印如下的输出：
```
MyObject::DosomethingMyObject::Dosomething, d:\documents\visual studio 2013\projects\demo\cppdemo\cppdemo.cpp, line 69
wmainwmain, d:\documents\visual studio 2013\projects\demo\cppdemo\cppdemo.cpp, line 147
__tmainCRTStartup__tmainCRTStartup, f:\dd\vctools\crt\crtw32\dllstuff\crtexe.c,line 623
wmainCRTStartupwmainCRTStartup, f:\dd\vctools\crt\crtw32\dllstuff\crtexe.c, line 466
BaseThreadInitThunk
RtlInitializeExceptionChain
RtlInitializeExceptionChain
Press any key to continue . . .
```


title: 如何调试msbuild？
date: 2015-04-15 23:34:28
categories:
tags: Debug
description: 本文介绍了如何调试msbuild，以及如何让msbuild编译solution文件sln。
---
有没有想过调试`msbuild`呢？可以参考[Debugging MSBuild script with Visual Studio](http://blogs.msdn.com/b/visualstudio/archive/2010/07/06/debugging-msbuild-script-with-visual-studio.aspx)，[part2](http://blogs.msdn.com/b/visualstudio/archive/2010/07/09/debugging-msbuild-script-with-visual-studio-2.aspx)，[part3](http://blogs.msdn.com/b/visualstudio/archive/2010/07/09/debugging-msbuild-script-with-visual-studio-3.aspx)。

#前期准备工作
1. 修改注册表，在键值`HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\MSBuild\4.0`下添加`String Value`，名字是** DebuggerEnabled**，值是**true**。（注意区分32位和64位，我给出的例子是32位的）  
2. 在命令行运行`msbuild /?`，可以看到多了一个`/debug`的选项，它的解释是：
>                     Causes a debugger prompt to appear immediately so that
>                     Visual Studio can be attached for you to debug the
>                     MSBuild XML and any tasks and loggers it uses.  
3. 确认Visual Studio是exception handler。就是确认注册表的`HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows NT\CurrentVersion\AeDebug\Debugger`的值是`vsjitdebugger.exe -p %ld -e %ld`。或者直接在命令行运行`vsjitdebugger.exe /regserver`。

#编译单个项目
在命令行运行`msbuild /t:rebuild /debug Demo.csproj`就会在刚刚启动`msbuild`时弹出Visual Studio的Attach窗口，接着就能F10，F11一步步的看`.csproj`这个文件是怎么执行的了，还可以在Locals窗口看到局部变量的值。

#编译整个solution
1. 生成`*.sln.metaproj`文件。  
因为`msbuild`缺省不支持编译`sln`，我们需要首先设置环境变量`set MSBUILDEMITSOLUTION=1`，这样用`msbuild`编译`sln`时就会生成一个`*.sln.metaproj`的文件。  
2. 在命令行运行`msbuild /t:rebuild /debug Demo.sln.metaproj`就可以编译整个solution了，可以在每一个项目文件的开头打上断点看编译是怎么执行的。

title: 在Visual Studio中使用Pseudovariables来帮助调试
date: 2015-07-15 07:23:23
categories:
tags: [Debug, CSharp, CPP]
description:
---
在我之前的博文[C#的强迫执行域Constrained Execution Regions(CERs)](/2013/06/07/constrained-execution-regions-in-csharp/)中提到过一点可以通过在Visual Studio的Watch窗口输入`@err,hr`来显示`GetLastError`。今天把Visual Studio中能使用的[Pseudovariables](https://msdn.microsoft.com/en-us/library/ms164891.aspx)都列一下，灵活运用这些Pseudovariables，可以大大的提高调试的效率。

在C++的程序中可以使用的Pseudovariables如下：

变量名|功能
---|---
`$err`|显示通过`SetLastError`设置的错误代码，何在代码中调用`GetLastError`效果一样。可以用`$err,hr`来显示错误代码对应的文本信息
`$handles`|显示应用程序中的句柄数量
`$vframe`|显示当前栈帧的地址
`$tid`|显示当前的线程ID
`$env`|显示当前应用的环境变量
`$cmdline`|显示运行当前应用的命令行参数
`$pid`|显示进程ID
`$registername`或`@registername`|显示寄存器的内容，如果寄存器的名字不和当前其他变量重名，直接写寄存器的名字也行
`$clk`|显示clock cycles
`$user`|显示运行当前用户的用户信息

在C#的程序中可以使用的Pseudovariables如下：

变量名|功能
---|---
`$exception`|显示最后的异常信息
`$user`|显示运行当前用户的用户信息，也适用于C#的应用

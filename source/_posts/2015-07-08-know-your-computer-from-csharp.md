title: 用C#来查看电脑硬件和系统信息
date: 2015-07-08 22:53:13
categories:
tags: CSharp
description: 本文介绍了如何用C#通过Environment和ManagementClass来查看操作系统，环境变量，CPU情况，硬盘情况等。
---
C#中有几个API可以很方便的查看电脑硬件和系统信息。
#`Environment`
下面的示例程序展示了从中[`Environment`](https://msdn.microsoft.com/en-us/library/System.Environment%28v=vs.110%29.aspx)我们能拿到哪些东西。比如  
1. `OSVersion`告诉我们操作系统的版本号。  
1. `GetLogicalDrives`可以告诉我们现在都有哪些盘符了。  
1. `Is64BitOperatingSystem`告诉我们是不是64位的系统。  
1. `ProcessorCount`告诉我们有几个CPU。  
1. `GetEnvironmentVariables`告诉我们现在的环境变量都有哪些，我们也可以只看进程的，用户的，机器的。  
1. `GetFolderPath`告诉我们[特殊文件夹（SpecialFolder）](https://msdn.microsoft.com/en-us/library/system.environment.specialfolder%28v=vs.110%29.aspx)的位置，包含`Desktop`、`Favorites`、`ProgramFilesX86`、`Recent`、`SystemX86`等。  
```
Console.WriteLine("------------------------------------------------------");
Console.WriteLine("From Environment:");
Console.WriteLine("------------------------------------------------------");
Console.WriteLine("CommandLine: " + Environment.CommandLine);
Console.WriteLine("CurrentDirectory: " + Environment.CurrentDirectory);
Console.WriteLine("CurrentManagedThreadId: " + Environment.CurrentManagedThreadId);
Console.WriteLine("ExitCode: " + Environment.ExitCode);
Console.WriteLine("LogicalDrives: " + string.Join(",", Environment.GetLogicalDrives()));
Console.WriteLine("HasShutdownStarted: " + Environment.HasShutdownStarted);
Console.WriteLine("Is64BitOperatingSystem: " + Environment.Is64BitOperatingSystem);
Console.WriteLine("Is64BitProcess: " + Environment.Is64BitProcess);
Console.WriteLine("MachineName: " + Environment.MachineName);
Console.WriteLine("OSVersion: " + Environment.OSVersion);
Console.WriteLine("ProcessorCount: " + Environment.ProcessorCount);
Console.WriteLine("StackTrace: " + Environment.StackTrace);
Console.WriteLine("SystemDirectory: " + Environment.SystemDirectory);
Console.WriteLine("SystemPageSize: " + Environment.SystemPageSize);
Console.WriteLine("TickCount: " + Environment.TickCount);
Console.WriteLine("UserDomainName: " + Environment.UserDomainName);
Console.WriteLine("UserInteractive: " + Environment.UserInteractive);
Console.WriteLine("UserName: " + Environment.UserName);
Console.WriteLine("Version: " + Environment.Version);
Console.WriteLine("WorkingSet: " + Environment.WorkingSet);
Console.WriteLine("Desktop: " + Environment.GetFolderPath(Environment.SpecialFolder.Desktop));
Console.WriteLine("EnvironmentVariables:");
foreach (DictionaryEntry envVar in Environment.GetEnvironmentVariables())
{
   Console.WriteLine("{0} : {1}", envVar.Key, envVar.Value);
}
```
#`ManagementClass`
[`ManagementClass`](https://msdn.microsoft.com/en-us/library/system.management.managementclass%28v=vs.110%29.aspx)可以告诉我们关于逻辑磁盘和CPU的更详细的信息。示例代码如下：

```
private static void Main(string[] args)
{
	OutputManagementClass("Win32_Processor");
	OutputManagementClass("Win32_LogicalDisk");
}
private static void OutputManagementClass(string path)
{
	Console.WriteLine("------------------------------------------------------");
	Console.WriteLine("From {0}", path);
	foreach (var obj in new ManagementClass(path).GetInstances())
	{
		foreach (var myProperty in new ManagementClass(path).Properties)
		{
			Console.WriteLine("{0}: {1}", myProperty.Name, obj.Properties[myProperty.Name].Value);
		}
	}
	Console.WriteLine();
}
```

磁盘方面，我们可以拿到关于磁盘总大小，剩余空间大小，文件系统等信息。

CPU方面，我们可以拿到主频，缓存大小，CPU物理核数和逻辑核数等信息。


title: 如何在一个进程开始运行时得到通知
date: 2015-09-17 19:49:24
categories:
tags: CSharp
description:
---
如果我们想在某个进程被执行时做一些事情，该怎么做呢？最简单粗暴的办法是来个死循环，不停地检查这个进程存不存在。代码如下所示：

```c#
private static void monitorProcessRuning(string name)
{
	while (true)
	{
		if (Process.GetProcessesByName(name).Length > 0)
		{
			Console.WriteLine("Process [{0}] is running", name);
		}
		Thread.Sleep(1000);
	}
}

static void Main(string[] args)
{
	monitorProcessRuning("notepad");
}
```

这个看起来技术含量有点低，那就试试别的办法。我们可以求助于[__InstanceCreationEvent](https://msdn.microsoft.com/en-us/library/aa394649%28v=vs.85%29.aspx)。当有一个新的实例生成时会触发这个事件。我们可以通过[WMI - Windows Management Instrumentation](https://msdn.microsoft.com/en-us/library/aa394582%28v=vs.85%29.aspx)来查询。如下是MSDN上给出的一个示例，当记事本程序运行时，会得到通知，使用了WMI中的[Win32_Process](https://msdn.microsoft.com/en-us/library/aa394372%28v=vs.85%29.aspx)。

```sql
SELECT * FROM __InstanceCreationEvent WITHIN PollingInterval WHERE TargetInstance ISA 'Win32_Process' and TargetInstance.Name = 'notepad.exe' 
```

下面来写我们的C#程序吧，需要用到[ManagementEventWatcher](https://msdn.microsoft.com/en-us/library/system.management.managementeventwatcher.aspx)。

```c#
private static void monitorProcessRuning(string name)
{
	WqlEventQuery processQuery = new WqlEventQuery("__InstanceCreationEvent", new TimeSpan(0, 0, 1),
		"TargetInstance isa 'Win32_Process' AND TargetInstance.Name = '" + name + "'");

	using (var watcher = new ManagementEventWatcher(processQuery))
	{
		watcher.EventArrived += (sender, eventArgs) =>
		{
			Console.WriteLine("Process [{0}] is running", name);
		};
		watcher.Start();

		Console.ReadLine();
		watcher.Stop();
	}
}

static void Main(string[] args)
{
	monitorProcessRuning("notepad.exe");
}
```

我们看看能不能用这个方法来解决之前的博文[Windows下如何检测用户修改了系统时间并且把系统时间改回来](/2015/09/16/how-to-change-time-back-after-user-change-time-in-windows/)，在那篇文章中是用户修改过系统时间后会得到通知。我们可以看看那个修改时间的进程是什么，然后就可以在那个进程启动时得到通知了。

首先通过Process Explorer看出启动的进程是`rundll32.exe`，命令行为`"C:\WINDOWS\system32\rundll32.exe" Shell32.dll,Control_RunDLL "C:\WINDOWS\system32\timedate.cpl",`。我们可以用下面的代码来监测这个进程启动，我们甚至可以通过监测到它启动就杀掉这个进程来阻止用户修改系统时间。

```c#
private static void monitorUserChangeTime(bool kill)
{
	var target = "rundll32.exe";
	string arg = "timedate.cpl";
	WqlEventQuery processQuery = new WqlEventQuery("__InstanceCreationEvent", new TimeSpan(0, 0, 1),
		"TargetInstance isa 'Win32_Process' AND TargetInstance.Name = '" + target + "'");

	using (var watcher = new ManagementEventWatcher(processQuery))
	{
		watcher.EventArrived += (sender, eventArgs) =>
		{
			var plist = Process.GetProcessesByName(Path.GetFileNameWithoutExtension(target));
			foreach (var p in plist)
			{
				using (var searcher = new ManagementObjectSearcher("SELECT CommandLine FROM Win32_Process WHERE ProcessId = " + p.Id))
				{
					var commandLine = new StringBuilder();
					foreach (var @object in searcher.Get())
					{
						commandLine.Append(@object["CommandLine"]);
						commandLine.Append(" ");
					}
					if (commandLine.ToString().Contains(arg))
					{
						if (kill)
						{
							Console.WriteLine("Killing the process!");
							p.Kill();
						}
						else
						{
							Console.WriteLine("User is changing the time!");
						} 
						
					}
				}
			}
		};
		watcher.Start();

		Console.ReadLine();
		watcher.Stop();
	}
}
```

注意上面的代码中获取进程命令行又是通过WMI实现的，使用了[ManagementObjectSearcher](https://msdn.microsoft.com/en-us/library/system.management.managementobjectsearcher(v=vs.110).aspx)从[Win32_Process](https://msdn.microsoft.com/en-us/library/aa394372%28v=vs.85%29.aspx)中获取命令行。

不能简单的通过[ProcessStartInfo.Arguments](https://msdn.microsoft.com/en-us/library/system.diagnostics.processstartinfo.arguments%28v=vs.110%29.aspx)来获取命令行。[ProcessStartInfo](https://msdn.microsoft.com/en-us/library/system.diagnostics.processstartinfo%28v=vs.110%29.aspx)只有在用`Process.Start`时才有用，在用`GetProcesses`拿到的进程中，`StartInfo`不包含进程的文件名和命令行参数。

> If you did not use the Start method to start a process, the StartInfo property does not reflect the parameters used to start the process. For example, if you use GetProcesses to get an array of processes running on the computer, the StartInfo property of each Process does not contain the original file name or arguments used to start the process.

title: 如何通过进程名获取进程ID
date: 2015-08-20 18:45:02
categories:
tags: Programming
description: 本文通过查看C#的GetProcessesByName的源代码来看如何在C++中获取所有系统进程信息。
---
之前的博客[C++如何拿到一个窗口的标题](/2015/08/19/how-to-get-the-window-title/)在最后总结如何获取窗口标题时的一个建议是只查询属于自己感兴趣的进程的窗口。Windows的函数[GetWindowThreadProcessId](https://msdn.microsoft.com/en-us/library/windows/desktop/ms633522%28v=vs.85%29.aspx)可以拿到一个窗口属于的进程ID，通常我们是知道我们感兴趣的进程名字，那么怎么根据进程名字来找到进程ID呢？

如果是C#那就非常简单了，直接使用[Process.GetProcessesByName](https://msdn.microsoft.com/en-us/library/z3w4xdc9%28v=vs.110%29.aspx)函数就能返回对应进程名字的进程数组。那如何在C++中实现呢？

老办法，翻翻C#的源代码（`Process.cs`）：

```csharp
public static Process[] GetProcessesByName(string processName) {
	return GetProcessesByName(processName, ".");
}

public static Process[] GetProcessesByName(string processName, string machineName) {
	if (processName == null) processName = String.Empty;
	Process[] procs = GetProcesses(machineName);
	ArrayList list = new ArrayList();

	for(int i = 0; i < procs.Length; i++) {                
		if( String.Equals(processName, procs[i].ProcessName, StringComparison.OrdinalIgnoreCase)) {
			list.Add( procs[i]);                    
		} else {
			procs[i].Dispose();
		}
	}
	
	Process[] temp = new Process[list.Count];
	list.CopyTo(temp, 0);
	return temp;
}
			
public static Process[] GetProcesses(string machineName) {
	bool isRemoteMachine = ProcessManager.IsRemoteMachine(machineName);
	ProcessInfo[] processInfos = ProcessManager.GetProcessInfos(machineName);
	Process[] processes = new Process[processInfos.Length];
	for (int i = 0; i < processInfos.Length; i++) {
		ProcessInfo processInfo = processInfos[i];
		processes[i] = new Process(machineName, isRemoteMachine, processInfo.processId, processInfo);
	}
	Debug.WriteLineIf(processTracing.TraceVerbose, "Process.GetProcesses(" + machineName + ")");
#if DEBUG
	if (processTracing.TraceVerbose) {
		Debug.Indent();
		for (int i = 0; i < processInfos.Length; i++) {
			Debug.WriteLine(processes[i].Id + ": " + processes[i].ProcessName);
		}
		Debug.Unindent();
	}
#endif
	return processes;
}
```

先拿到所有的进程，然后比较进程名字。`Process.cs`调用了`ProcessManager.cs`来拿所有的进程，接着看源代码（`ProcessManager.cs`）：

```csharp
public static ProcessInfo[] GetProcessInfos(string machineName) {
	bool isRemoteMachine = IsRemoteMachine(machineName);
	if (IsNt) {
		// Do not use performance counter for local machine with Win2000 and above
		if( !isRemoteMachine && 
			(Environment.OSVersion.Version.Major >= 5 ))   {
			return NtProcessInfoHelper.GetProcessInfos();
		}
		return NtProcessManager.GetProcessInfos(machineName, isRemoteMachine);
	}

	else {
		if (isRemoteMachine)
			throw new PlatformNotSupportedException(SR.GetString(SR.WinNTRequiredForRemote));
		return WinProcessManager.GetProcessInfos();
	}
}
```

接着往下看就能看到C#是如何取进程ID的了。有下面几种实现：

1. Windows 2000以上的系统用`NtProcessInfoHelper.GetProcessInfos()`。实现就是使用[NtQuerySystemInformation](https://msdn.microsoft.com/en-us/library/windows/desktop/ms724509%28v=vs.85%29.aspx)函数，根据参数`SYSTEM_PROCESS_INFORMATION`，可以获取包含所有进程信息的一个数组。  
1. Windows NT以上Windows2000以下用`NtProcessManager.GetProcessInfos`。实现就是通过`PerformanceCounterLib`来获取进程数组。  
1. Windows NT之前的用`WinProcessManager.GetProcessInfos()`。实现就是使用[CreateToolhelp32Snapshot](https://msdn.microsoft.com/en-us/library/windows/desktop/ms682489%28v=vs.85%29.aspx)函数，根据参数`TH32CS_SNAPPROCESS`来包含所有进程，然后根据[Process32First](https://msdn.microsoft.com/en-us/library/windows/desktop/ms684834%28v=vs.85%29.aspx)来遍历进程。  


下面是一个简化版的用`CreateToolhelp32Snapshot`来获取给定进程名字进程ID的例子。

```cpp
vector<DWORD> GetProcessIDByName(CComBSTR processName)
{
    vector<DWORD> processIDs;

    PROCESSENTRY32 entry;
    entry.dwSize = sizeof(PROCESSENTRY32);

    HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, NULL);
    if (Process32First(snapshot, &entry) == TRUE)
    {
        while (Process32Next(snapshot, &entry) == TRUE)
        {
            if (CComBSTR(entry.szExeFile) == processName)
            {
                processIDs.push_back(entry.th32ProcessID);
            }
        }
    }
    CloseHandle(snapshot);

    return processIDs;
}
```

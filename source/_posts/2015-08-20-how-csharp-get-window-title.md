title: C#是怎么获取窗口标题的
date: 2015-08-20 09:29:52
categories:
tags: CSharp
description:
---
之前的博客[C++如何拿到一个窗口的标题](/2015/08/19/how-to-get-the-window-title/)介绍了如何用Windows的GetWindowText函数来获取窗口的标题，Windows是如何实现这个方法的，同时介绍了在使用GetWindowText时可能遇到的一些问题。下面我们来看看在C#中是怎么拿窗口标题的。

C#的[Process.MainWindowTitle](https://msdn.microsoft.com/en-us/library/system.diagnostics.process.mainwindowtitle%28v=vs.110%29.aspx)可以获取进程主窗口的标题，如果进程没有UI，那么返回空字符串。如果进程刚启动，可以调用`WaitForInputIdle`来等待进程初始化完成。下面的程序代码可以遍历所有进程，打印主窗口标题。

```csharp
static void Main(string[] args)
{
	foreach (var p in Process.GetProcesses())
	{
		Console.WriteLine(p.MainWindowTitle);
	}
}
```

C#是怎么实现的呢？可以看看C#的源代码（`Process.cs`）：

```csharp
public class Process : Component {
	public string MainWindowTitle {
		[ResourceExposure(ResourceScope.None)]
		[ResourceConsumption(ResourceScope.Machine, ResourceScope.Machine)]
		get {
			if (mainWindowTitle == null) {
				IntPtr handle = MainWindowHandle;
				if (handle == (IntPtr)0) {
					mainWindowTitle = String.Empty;
				}
				else {
					int length = NativeMethods.GetWindowTextLength(new HandleRef(this, handle)) * 2;
					StringBuilder builder = new StringBuilder(length);
					NativeMethods.GetWindowText(new HandleRef(this, handle), builder, builder.Capacity);
					mainWindowTitle = builder.ToString();
				}
			}
			return mainWindowTitle;
		}
	}
}
```

我们看到也是调用了windows的`GetWindowText`方法，那么这个`MainWindowHandle`是怎么来的呢？接着翻代码（`Process.cs`）：

```csharp
public class Process : Component {
	public IntPtr MainWindowHandle {
		[ResourceExposure(ResourceScope.Machine)]
		[ResourceConsumption(ResourceScope.Machine)]
		get {
			if (!haveMainWindow) {
				EnsureState(State.IsLocal | State.HaveId);
				mainWindowHandle = ProcessManager.GetMainWindowHandle(processId);

				if (mainWindowHandle != (IntPtr)0) {
					haveMainWindow = true;
				} else {
					// We do the following only for the side-effect that it will throw when if the process no longer exists on the system.  In Whidbey
					// we always did this check but have now changed it to just require a ProcessId. In the case where someone has called Refresh() 
					// and the process has exited this call will throw an exception where as the above code would return 0 as the handle.
					EnsureState(State.HaveProcessInfo);
				}
			}
			return mainWindowHandle;
		}
	}
}
```

是通过`ProcessManager.GetMainWindowHandle`函数根据`processId`找到的，那再看看`ProcessManager.GetMainWindowHandle`的实现（`ProcessManager.cs`）：

```csharp
internal static class ProcessManager {
	public static IntPtr GetMainWindowHandle(int processId) {
		MainWindowFinder finder = new MainWindowFinder();
		return finder.FindMainWindow(processId);
	}
}

internal class MainWindowFinder {
	public IntPtr FindMainWindow(int processId) {
		bestHandle = (IntPtr)0;
		this.processId = processId;
		
		NativeMethods.EnumThreadWindowsCallback callback = new NativeMethods.EnumThreadWindowsCallback(this.EnumWindowsCallback);
		NativeMethods.EnumWindows(callback, IntPtr.Zero);

		GC.KeepAlive(callback);
		return bestHandle;
	}
	
	bool EnumWindowsCallback(IntPtr handle, IntPtr extraParameter) {
		int processId;
		NativeMethods.GetWindowThreadProcessId(new HandleRef(this, handle), out processId);
		if (processId == this.processId) {
			if (IsMainWindow(handle)) {
				bestHandle = handle;
				return false;
			}
		}
		return true;
	}	
	
	bool IsMainWindow(IntPtr handle) {
		
		if (NativeMethods.GetWindow(new HandleRef(this, handle), NativeMethods.GW_OWNER) != (IntPtr)0 || !NativeMethods.IsWindowVisible(new HandleRef(this, handle)))
			return false;
		
		// [....]: should we use no window title to mean not a main window? (task man does)
		
		/*
		int length = NativeMethods.GetWindowTextLength(handle) * 2;
		StringBuilder builder = new StringBuilder(length);
		if (NativeMethods.GetWindowText(handle, builder, builder.Capacity) == 0)
			return false;
		if (builder.ToString() == string.Empty)
			return false;
		*/

		return true;
	}	
}
```

这就清楚的看出来C#是怎么拿窗口标题了：

1. 根据`processId`使用`EnumWindows`方法找到属于当前进程的窗口。  
1. 根据窗口有没有GW_OWNER并且可见来判断是不是主窗口。  
1. 根据主窗口句柄使用`GetWindowText`方法得到窗口的标题。

跟我在博文中[C++如何拿到一个窗口的标题](/2015/08/19/how-to-get-the-window-title/)推荐的C++的使用方式一样。

如果在C#中想得到非主窗口的标题，可以用p/invoke来直接调用`EnumWindows`和`GetWindowText`。
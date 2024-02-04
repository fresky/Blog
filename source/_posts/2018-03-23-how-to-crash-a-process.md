title: 如何在一个进程中注入崩溃
date: 2018-03-23 23:21:22
categories:
tags: [Programming, Debug]
description:
---
我们在设计一个系统时经常需要考虑如果某个进程崩溃之后系统应该作何反应。可以用[force any running process to crash](https://stackoverflow.com/questions/10686231/force-any-running-process-to-crash)中介绍的方法在一个进程中注入一个异常导致它崩溃。

这个方法使用了[`CreateRemoteThread`](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createremotethread)API。`CreateRemoteThread`可以在目标进程内创建一个线程来执行一段代码，只要这段代码能够crash，我们就能做到在目标进程中注入一个崩溃了。下面是根据这个方法实现的C#的代码。

```csharp
class CrashInjecter
{
    [Flags]
    public enum ProcessAccessFlags : uint
    {
        All = 0x001F0FFF,
        Terminate = 0x00000001,
        CreateThread = 0x00000002,
        VirtualMemoryOperation = 0x00000008,
        VirtualMemoryRead = 0x00000010,
        VirtualMemoryWrite = 0x00000020,
        DuplicateHandle = 0x00000040,
        CreateProcess = 0x000000080,
        SetQuota = 0x00000100,
        SetInformation = 0x00000200,
        QueryInformation = 0x00000400,
        QueryLimitedInformation = 0x00001000,
        Synchronize = 0x00100000
    }

    [DllImport("kernel32.dll", SetLastError = true)]
    public static extern IntPtr OpenProcess(
        ProcessAccessFlags processAccess,
        bool bInheritHandle,
        int processId
    );

    [DllImport("kernel32.dll")]
    static extern IntPtr CreateRemoteThread(IntPtr hProcess,
        IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress,
        IntPtr lpParameter, uint dwCreationFlags, out IntPtr lpThreadId);

    public static void Crash(int processId)
    {
        var hProcess = OpenProcess(ProcessAccessFlags.All, false, processId);
        CreateRemoteThread(hProcess, IntPtr.Zero, 0, IntPtr.Zero, IntPtr.Zero, 0, out _);
    }
}
```
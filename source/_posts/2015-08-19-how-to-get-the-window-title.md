title: C++如何拿到一个窗口的标题
date: 2015-08-19 18:38:17
categories:
tags: [CPP, Debug]
description: 本文介绍了如何用Windows的GetWindowText函数来获取窗口的标题。介绍了Windows是如何实现这个方法的。同时介绍了在使用GetWindowText时可能遇到的一些问题，比如调用方被挂起hang了，原因是什么，如何避免。另外还说明了使用GetWindow函数可能遇到的问题，应该尽量使用EnumWindows来替代。
---
如果想拿到一个窗口的标题，那么使用Windows的函数[GetWindowText](https://msdn.microsoft.com/en-us/library/windows/desktop/ms633520%28v=vs.85%29.aspx)，但是使用这个函数时有一些坑需要注意一下。先看看MSDN上的说明：

> If the target window is owned by the current process, GetWindowText causes a WM_GETTEXT message to be sent to the specified window or control. If the target window is owned by another process and has a caption, GetWindowText retrieves the window caption text. If the window does not have a caption, the return value is a null string. This behavior is by design. It allows applications to call GetWindowText without becoming unresponsive if the process that owns the target window is not responding. However, if the target window is not responding and it belongs to the calling application, GetWindowText will cause the calling application to become unresponsive.

> To retrieve the text of a control in another process, send a WM_GETTEXT message directly instead of calling GetWindowText. 

这个说明写的不太好懂！可以参看The Old New Thing的博客[The secret life of GetWindowText](http://blogs.msdn.com/b/oldnewthing/archive/2003/08/21/54675.aspx)，说的更明白一些，下面我就简要介绍一下。

# GetWindowText是怎么工作的

1. 如果你要拿标题的窗口属于当前的进程，那么`GetWindowText`就是发送`WM_GETTEXT`消息过去，然后获取标题。
2. 如果你要拿标题的窗口不属于当前的进程，那么`GetWindowText`就不发送`WM_GETTEXT`消息，而是直接从一个特定的地方读取标题。

# GetWindowText为什么这么实现

窗口的标题有两种处理的方式：  
1. 让系统来处理，`WM_NCCREATE`消息会把`CreateWindow/Ex`的参数`lpWindowName`放到一个特定位置。处理`WM_GETTEXT`消息是就把这个特定位置存储的字符串返回出去。处理`WM_SETTEXT`消息就是把传进来的字符串放到那个特定位置。  
1. 窗口自己处理，自己决定如何响应`WM_GETTEXT`和`WM_SETTEXT`消息。

通常来说Frame窗口让系统来处理的，自定义的控件自己管理。

如果我们用发送`WM_GETTEXT`消息的方式来获取标题，会有一个潜在的风险，如果目标窗口已经没有反应了，那么就不能处理这个`WM_GETTEXT`消息，而获取方用的是send的方式，所以也会变得没有反应，就被刮起了。

所以Windows选择的的实现方式就是如果你要获取其他进程的窗口标题，那么就直接从特定位置取，这样可以保证调用方不会挂起。如果你要获取自己进程的窗口标题，那么就发送`WM_GETTEXT`消息，这样可以保证就算是窗口自己管理标题，也能拿到正确的值。当然，带来的风险就是假设这个窗口没有响应，那么调用方也就被挂起了，因为这个窗口是自己进程，挂起也没啥不应该的：）。

# 验证GetWindowText是怎么工作的

虽然我们没有`GetWindowText`的源代码，但是我们还是可以很容易的用调试的方式来验证它是不是这么实现的。先写如下的示例代码。

## `PrintProcessNameAndID`打印给定进程ID的进程名。

```cpp
void PrintProcessNameAndID(DWORD processID)
{
    TCHAR szProcessName[MAX_PATH] = TEXT("<unknown>");

    HANDLE hProcess = OpenProcess(PROCESS_QUERY_INFORMATION |
        PROCESS_VM_READ,
        FALSE, processID);

    if (NULL != hProcess)
    {
        HMODULE hMod;
        DWORD cbNeeded;

        if (EnumProcessModules(hProcess, &hMod, sizeof(hMod),
            &cbNeeded))
        {
            GetModuleBaseName(hProcess, hMod, szProcessName,
                sizeof(szProcessName) / sizeof(TCHAR));
        }
    }

    CString msg;
    msg.Format(L"%s(PID: %u)  ", szProcessName, processID);
    OutputDebugString(msg);

    CloseHandle(hProcess);
}
```

## `LoopWindow`遍历窗口并打印窗口标题。

```cpp
void LoopWindow()
{
    HWND hDesktopWnd = ::GetDesktopWindow();

    HWND hWindow = ::GetWindow(hDesktopWnd, GW_CHILD);

    while (hWindow != NULL)
    {
        DWORD processID;
        GetWindowThreadProcessId(hWindow, &processID);
        PrintProcessNameAndID(processID);

        int lTextLen = ::GetWindowTextLength(hWindow);
        CString str;
        ::GetWindowText(hWindow, str.GetBufferSetLength(lTextLen + 1), lTextLen + 1);

        OutputDebugString(str);
        OutputDebugString(L"\r\n");
        hWindow = ::GetWindow(hWindow, GW_HWNDNEXT);
    }
}
```

我们在主函数调用`LoopWindow`，就可以在debugger中看到所有窗口的标题输出。

```cpp
int _tmain(int argc, _TCHAR* argv[])
{
    LoopWindow();
	return 0;
}
```

## 用windbg来验证`GetWindowText`的行为

我们用windbg来运行编译好的exe，在发送消息的地方加上断点，如下所示。

```
bp user32!NtUserMessageCall
```

然后运行，我们就可以发现只有在获取当前进程的窗口时，这个断点才会进来。这就验证了如果你要拿标题的窗口属于当前的进程，那么`GetWindowText`就是发送`WM_GETTEXT`消息过去，然后获取标题。如果你要拿标题的窗口不属于当前的进程，那么`GetWindowText`就不发送`WM_GETTEXT`消息，而是直接从一个特定的地方读取标题。

# 模拟目标窗口无响应的情形。

## 目标窗口是其他进程的。

这个非常简单，随便用C#写一个winform，在一个按钮响应里写一个`while(true){}`，第一点这个按钮，窗口就没有相应了。我们运行上面的那个`LoopWindow`，发现没有任何问题，正常退出，窗口标题也能拿到。

## 目标窗口是当前进程的。

我们可以写一个MFC的程序，然后在第一个对话框中加两个按钮，第一个按钮会起一个线程不停的调用`LoopWindow`，第二个按钮会打开另一个对框框。第二个对话框有一个按钮，这个按钮的响应函数也是`while(true){}`。这样我们发现在点了第二个对话框的按钮之后，那个调用`LoopWindow`的线程也被挂起了。

# `GetWindow`带来的坑

我们的`LoopWindow`里使用了[GetWindow](https://msdn.microsoft.com/en-gb/library/windows/desktop/ms633515.asphttps://msdn.microsoft.com/en-gb/library/windows/desktop/ms633515.aspx)来寻找所有的窗口，但是这个`GetWindow`也有一个坑，MSDN上的说明是这么写的：

> The EnumChildWindows function is more reliable than calling GetWindow in a loop. An application that calls GetWindow to perform this task risks being caught in an infinite loop or referencing a handle to a window that has been destroyed. 

就是说在一个循环中调用`GetWindow`有可能会进入一个死循环。或者`GetWindow`有可能会返回一个已经被析构掉的一个窗口。

# 如何绕过这些坑

1. 遍历窗口要使用[EnumWindows](https://msdn.microsoft.com/en-gb/library/windows/desktop/ms633497.aspx)或者[EnumChildWindows](https://msdn.microsoft.com/en-gb/library/windows/desktop/ms633494.aspx)。  
1. 如果我们不希望调用`GetWindowText`的线程挂起，需要做超时处理。  
1. 大部分情况下我们获取窗口标题并不需要处理当前进程的窗口，所以在调用`GetWindowText`之前可以做个判断，如果是当前进程就直接返回。如果我们只需要获取某些特定进程的窗口标题，可以在调用`GetWindowText`之前判断目标窗口是否属于我们感兴趣的进程。  













































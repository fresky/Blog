title: 如何去掉WinForm或者WPF的最大化和最小化按钮
date: 2015-07-14 23:13:18
categories:
tags: CSharp
description: 本文介绍了如何通过SetWindowLongPtr和GetWindowLongPtr、SetWindowPos来去掉WinForm或者WPF的最大化和最小化按钮。
---
有时候我们希望对我们的WinForm或者WPF控件做一些定制，比如去掉最大化和最小化按钮，怎么做呢？

# WinForm
WinForm去掉最大化和最小化按钮：
```csharp
MaximizeBox = false;
MinimizeBox = false;
```

WinForm的最大化和最小化按钮和能否改变窗口大小是两个属性，所以上面的代码只是把最小化和最大化按钮去掉了，但是窗口还能改变大小。

下面的代码是不允许改变WinForm的窗口大小，但是最大化和最小化按钮还是能点的。
```csharp
FormBorderStyle = FormBorderStyle.FixedSingle;
```

# WPF
WPF没有API可以直接去掉最大化和最小化按钮的，但是我们可以通过`ResizeMode`来曲线救国，如果设置窗口不能改变大小，那么最大化和最小化按钮也就没有了，如下所示：
```csharp
ResizeMode = System.Windows.ResizeMode.NoResize;
```

另外还有一个办法是通过设置`WindowStyle`来实现，但是这个会把关闭的按钮也弄没，如下所示：
```csharp
WindowStyle = WindowStyle.None;
```

这两种方式都不是很好，会有一些副作用，那还有别的办法吗？

# 用Windows的API：SetWindowLongPtr
[SetWindowLongPtr](https://msdn.microsoft.com/en-us/library/windows/desktop/ms644898%28v=vs.85%29.aspx)是Windows的一个API，作用是改变窗口的属性。函数签名如下：
```c++
LONG_PTR WINAPI SetWindowLongPtr(
  _In_ HWND     hWnd,
  _In_ int      nIndex,
  _In_ LONG_PTR dwNewLong
);
```
第一个参数是窗口句柄。第二个参数是offset，当它的值是`GWL_STYLE`（-16）时，可以设置窗口风格。第三个参数是要设置的值，比如针对`GWL_STYLE`的可能取值参见[Window Styles](https://msdn.microsoft.com/en-us/library/windows/desktop/ms632600%28v=vs.85%29.aspx)。里面就有最大化（`WS_MAXIMIZEBOX=0x00010000L`）和最小化按钮（`WS_MINIMIZEBOX=0x00020000L`）。所以我们可以通过设置这两个值去掉最大化和最小化按钮。

在调用[SetWindowLongPtr](https://msdn.microsoft.com/en-us/library/windows/desktop/ms644898%28v=vs.85%29.aspx)之前要调用[GetWindowLongPtr](https://msdn.microsoft.com/en-us/library/windows/desktop/ms633585%28v=vs.85%29.aspx)来获取当前Window的信息，确保我们只是把最大化和最小化按钮去了。

在调用[SetWindowLongPtr](https://msdn.microsoft.com/en-us/library/windows/desktop/ms644898%28v=vs.85%29.aspx)之前要调用[SetWindowPos](https://msdn.microsoft.com/en-us/library/windows/desktop/ms633545%28v=vs.85%29.aspx)以确保我们的设置生效了。
[SetWindowLongPtr](https://msdn.microsoft.com/en-us/library/windows/desktop/ms644898%28v=vs.85%29.aspx)和[GetWindowLongPtr](https://msdn.microsoft.com/en-us/library/windows/desktop/ms633585%28v=vs.85%29.aspx)都是**针对64位程序**的。

尽管MSDN上写的是这两个对32位也有效，但是其实如果直接`DllImport`这两个API使用时会报`EntryPointNotFoundException`。所以我们要根据当前应用程序是32位还是64位使用不同的`DllImport`API。**32位**下需要用[GetWindowLong](https://msdn.microsoft.com/en-us/library/windows/desktop/ms633584%28v=vs.85%29.aspx)和[SetWindowLong](https://msdn.microsoft.com/en-us/library/windows/desktop/ms633591%28v=vs.85%29.aspx)。

下面的代码对WinForm和WPF都有效：
```csharp
private const int GWL_STYLE = -16;
private const int WS_MAXIMIZEBOX = 0x10000;
private const int WS_MINIMIZEBOX = 0x20000;
private const int SWP_NOMOVE = 0x0002;
private const int SWP_NOSIZE = 0x0001;
private const int SWP_NOZORDER = 0x0004;
private const int SWP_FRAMECHANGED = 0x0020;

[DllImport("user32.dll", EntryPoint = "GetWindowLong", CharSet = CharSet.Auto)]
private static extern int GetWindowLong32(IntPtr hWnd, int nIndex);

[DllImport("user32.dll")]
extern private static int GetWindowLongPtr(IntPtr hwnd, int index);

[DllImport("user32.dll", EntryPoint = "SetWindowLong", CharSet = CharSet.Auto)]
extern private static int SetWindowLong32(IntPtr hwnd, int index, int value);

[DllImport("user32.dll")]
extern private static int SetWindowLongPtr(IntPtr hwnd, int index, int value);

[DllImport("user32.dll")]
public static extern bool SetWindowPos(IntPtr hWnd, int hWndInsertAfter, int x, int Y, int cx, int cy, int wFlags);

private static void HideMinMaxButtons(IntPtr hwnd)
{
	if (IntPtr.Size == 4)
	{
		var currentStyle = GetWindowLong32(hwnd, GWL_STYLE);
		SetWindowLong32(hwnd, GWL_STYLE, (currentStyle & ~WS_MAXIMIZEBOX & ~WS_MINIMIZEBOX));
	}
	else
	{
		var currentStyle = GetWindowLongPtr(hwnd, GWL_STYLE);
		SetWindowLongPtr(hwnd, GWL_STYLE, (currentStyle & ~WS_MAXIMIZEBOX & ~WS_MINIMIZEBOX));
	}
	//call SetWindowPos to make sure the SetWindowLongPtr take effect according to MSDN
	SetWindowPos(hwnd, 0, 0, 0, 0, 0, SWP_NOMOVE | SWP_NOSIZE | SWP_NOZORDER | SWP_FRAMECHANGED);
}
```

另外要注意的一点事，在调用过`HideMinMaxButtons`之后不能在调用其他会改变窗口风格的C#的API。比如如果WinForm，在这个函数之后在调用一下`FormBorderStyle = FormBorderStyle.Sizable;`，或者WPF调用一下`ResizeMode = System.Windows.ResizeMode.CanResize;`，那么最大化和最小化按钮就又回来了。



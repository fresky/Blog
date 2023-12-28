title: 为什么只有在用Visual Studio启动程序时会抛出InvalidOperationException异常
date: 2015-09-14 18:22:24
categories:
tags: [Programming]
description:
---
在我之前的博文[C#中只使用Invokerequired来判断是不是UI线程可靠吗？](/2014/05/14/is-invokerequired-reliable/)提到过除了检查`InvokeRequired`，还需要检查`IsHandleCreated`。今天再分享一段关于`InvokeRequired`的代码。

```
public partial class Form9 : Form
{
	public Form9()
	{
		InitializeComponent();
		// CheckForIllegalCrossThreadCalls = false;
	}

	private void button1_Click(object sender, EventArgs e)
	{
		var t = new Thread(() => { textBox1.Text = "Random Text"; });
		t.Start();
	}
}
```	

这段代码很简单，就是在非UI线程更新UI控件。如果用Visual Studio启动程序时会抛出`InvalidOperationException`异常。但是如果直接双击启动编译后的exe文件，则一切工作顺利，没有InvalidOperation异常抛出。就算双击exe文件启动后拿Visual Studio Attach到这个进程，设置Visual Studio在所有的异常抛出时都break，还是没有抓到。为什么双击exe文件就神奇的工作了呢？

回头看MSDN的[How to: Make Thread-Safe Calls to Windows Forms Controls](https://msdn.microsoft.com/en-us/library/ms171728.aspx)。

> Access to Windows Forms controls is not inherently thread safe. If you have two or more threads manipulating the state of a control, it is possible to force the control into an inconsistent state. Other thread-related bugs are possible, such as race conditions and deadlocks. It is important to make sure that access to your controls is performed in a thread-safe way.

> **It is unsafe to call a control from a thread other than the one that created the control without using the Invoke method. **

这段写的就是说不能跨线程操作UI控件。接着往下读。

> The .NET Framework helps you detect when you are accessing your controls in a manner that is not thread safe. When you are running your application in the debugger, and a thread other than the one which created a control tries to call that control, the debugger raises an InvalidOperationException with the message, "Control control name accessed from a thread other than the thread it was created on."

> **This exception occurs reliably during debugging and, under some circumstances, at run time.** You might see this exception when you debug applications that you wrote with the .NET Framework prior to the .NET Framework 2.0. **You are strongly advised to fix this problem when you see it, but you can disable it by setting the CheckForIllegalCrossThreadCalls property to false.** 

这段写的就是说如果是用Visual Studio运行程序，会强制抛出一个`InvalidOperationException`，**帮助我们发现这些问题然后修复他们**。如果没有通过Visual Studio直接运行程序，不会稳定的抛出这个异常，但是并不是说没有问题。

另外，如果暂时不想修改代码，但是还想从Visual Studio中启动，可以在Form中设置属性[CheckForIllegalCrossThreadCalls](https://msdn.microsoft.com/en-us/library/system.windows.forms.control.checkforillegalcrossthreadcalls%28v=vs.80%29.aspx)为false，这样Visual Studio就会忽略跨线程访问UI的情况，不抛出`InvalidOperationException`异常了。就是我上面列的代码注释掉的那行。`CheckForIllegalCrossThreadCalls`是.NET Framework 2.0引入的一个新属性。
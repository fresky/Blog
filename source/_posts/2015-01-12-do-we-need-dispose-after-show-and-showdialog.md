title: 你知道Form.Show()和Form.ShowDialog()的区别吗？
date: 2015-01-12 20:21:16
categories:
tags: CSharp
description: 本文解析了Form.Show()和Form.ShowDialog()的区别，简单的说，就是调用Form.Show()不需要显示的Dispose，但是调用Form.ShowDialog()需要显示的Dispose。
---

在我的博文[来试试这个来自静态代码分析工具PVS Studio提供C++的小测验吧](/2014/12/22/cpp-quiz-from-pvs-studio/)提到过静态检查工具可以帮助我们找到很多的编程问题，但是在修改这些小问题时也要小心，弄清楚问题的来龙去脉，不能随便凭感觉改，否则会引入新的问题，最近就遇到这样的一个例子。

###问题描述——`Form.Show`没有`Dispose`的警告
先看示例代码吧，很简单，就是在点击一个按钮时弹出一个`Form`。Coverity报错，说这个`Form2`创建之后在出作用域之前没有`Dispose`。

```c#
private void button1_Click(object sender, EventArgs e)
{
	Form2 f = new Form2();
	f.Show();
}
```

####第一次修改

既然没有`Dispose`，那么标准Fix当然是使用`Using`。于是第一个修改就诞生了，如下所示：

```c#
private void button1_Click(object sender, EventArgs e)
{
	using (Form2 f = new Form2())
	{
		f.Show();
	}
}
```

嗯，Coverity不再报错了，可是一运行代码发现，不对啊，怎么这个对话框闪退呢……

原因很清晰了，打开了一个`Form`，但是紧接着就把它`Dispose`掉了，当然就关掉了。呵呵，想起了我曾经写过的博客[谁动了我的timer？C#的垃圾回收和调试](/2013/06/20/where-is-my-timer-csharp-gc/)，是不是有点异曲同工：）

####这是Coverity警告是对的吗？
我觉得这个Coeverity警告是一个False Positive，因为Form调用完Show()之后用户关掉时会调用Dispose，具体可以参见[MSDN](http://msdn.microsoft.com/en-us/library/system.windows.forms.control.show%28v=vs.110%29.aspx)的示例代码。

###再来看看`Form.ShowDialog`有没有什么不同
假如我们用的是模态对话框，那么代码是下面这样的：
```c#
private void button1_Click(object sender, EventArgs e)
{
	Form2 f = new Form2();
	f.ShowDialog();
}
```

那估计Coverity还是会报一个警告，说创建完对象没有`Dispose`，那么这个警告是False Positive吗？继续翻[MSDN](http://msdn.microsoft.com/en-us/library/c7ykbedk.aspx)吧，写着如下说明：

> When a form is displayed as a modal dialog box, clicking the Close button (the button with an X at the upper-right corner of the form) causes the form to be hidden and the DialogResult property to be set to DialogResult.Cancel. **Unlike non-modal forms**, the Close method is not called by the .NET Framework when the user clicks the close form button of a dialog box or sets the value of the DialogResult property. Instead the form is hidden and can be shown again without creating a new instance of the dialog box. Because a form displayed as a dialog box is hidden instead of closed, you **must call the Dispose method** of the form when the form is no longer needed by your application.

MSDN上的示例代码如下：
```c#
public void ShowMyDialogBox()
{
   Form2 testDialog = new Form2();

   // Show testDialog as a modal dialog and determine if DialogResult = OK. 
   if (testDialog.ShowDialog(this) == DialogResult.OK)
   {
      // Read the contents of testDialog's TextBox. 
      this.txtResult.Text = testDialog.TextBox1.Text;
   }
   else
   {
      this.txtResult.Text = "Cancelled";
   }
   testDialog.Dispose();
}
```

所以如果用`ShowDialog`，就需要这样写了：
```c#
private void button1_Click(object sender, EventArgs e)
{
	using (Form2 f = new Form2())
	{
		f.ShowDialog();
	}
}
```

###试试GC能帮什么忙吧
虽然`ShowDialog()`不能`Dispose`，但是因为这个Form是个局部变量，出了作用域应该就可以被回收了吧，我们试试看强制调用`GC.Collect()`会怎样。于是我加了个按钮，就是去强制垃圾回收，一切符合预期，这个`Form`确实被`Dispose`掉了。

那回过头来再试试`Show()`，假如我这样写：
```c#
private void button1_Click(object sender, EventArgs e)
{
	Form2 f = new Form2();
	f.Show();
	f.Hide();
}
```

那么这个窗口还是会闪退，出了这个作用域，调用GC试试看吧。Oops！！！`Dispose`没有被调到，什么情况？这个Form究竟还被谁引用这呢？只好祭出windbg了，attach，然后敲`!gcroot`命令，得到了如下的输出：

```
0:005> !gcroot 02241e6c 
HandleTable:
    000a13e8 (pinned handle)
    -> 03205390 System.Object[]
    -> 02227480 System.Collections.Generic.Dictionary`2[[System.Object, mscorlib],[System.Collections.Generic.List`1[[Microsoft.Win32.SystemEvents+SystemEventInvokeInfo, System]], mscorlib]]
    -> 02227a30 System.Collections.Generic.Dictionary`2+Entry[[System.Object, mscorlib],[System.Collections.Generic.List`1[[Microsoft.Win32.SystemEvents+SystemEventInvokeInfo, System]], mscorlib]][]
    -> 02228fb4 System.Collections.Generic.List`1[[Microsoft.Win32.SystemEvents+SystemEventInvokeInfo, System]]
    -> 02228fdc System.Object[]
    -> 02242354 Microsoft.Win32.SystemEvents+SystemEventInvokeInfo
    -> 02242334 Microsoft.Win32.UserPreferenceChangedEventHandler
    -> 02241e6c Form1.Form2

    000a13ec (pinned handle)
    -> 03203390 System.Object[]
    -> 0222d7a4 System.Windows.Forms.FormCollection
    -> 0222d7bc System.Collections.ArrayList
    -> 0222d7d4 System.Object[]
    -> 02241e6c Form1.Form2

Found 2 unique roots (run '!GCRoot -all' to see all roots).
```

这个`UserPreferenceChangedEventHandler`又是什么呢？它是一个静态的系统事件，具体参见StackOverflow上的这个[回答](http://stackoverflow.com/a/1147729/304115)。因为是静态的，所以这个`Form`还被引用，所以不能释放。

那问题又来了，为啥`ShowDialog`可以被垃圾回收呢？翻开C#的源代码，找到`ShowDialog`方法，可以看到最后如下的代码：

```c#
finally {
	//...
	if (IsHandleCreated) {
		// ...
		DestroyHandle();
	}
	//...
}
```

这个会调到`Control`的`OnHandleDestroyed`，然后会注销事件。

```c#
protected virtual void OnHandleDestroyed(EventArgs e) {
	//...
	if (!RecreatingHandle) {
		//...
		ListenToUserPreferenceChanged(false /*listen*/);
	}
}
private void ListenToUserPreferenceChanged(bool listen) {
	if (GetState2(STATE2_LISTENINGTOUSERPREFERENCECHANGED)) {
		if (!listen) {
			SetState2(STATE2_LISTENINGTOUSERPREFERENCECHANGED, false);
			SystemEvents.UserPreferenceChanged -= new UserPreferenceChangedEventHandler(this.UserPreferenceChanged);
		}
	}
	else if (listen) {
		SetState2(STATE2_LISTENINGTOUSERPREFERENCECHANGED, true);
		SystemEvents.UserPreferenceChanged += new UserPreferenceChangedEventHandler(this.UserPreferenceChanged);
	}
}	
```

嗯，绕了一大圈，你搞明白`Form.Show()`和`Form.ShowDialog()`的区别了吗？简单的说，就是调用`Form.Show()`不需要显示的`Dispose`，但是调用`Form.ShowDialog()`需要显示的`Dispose`。


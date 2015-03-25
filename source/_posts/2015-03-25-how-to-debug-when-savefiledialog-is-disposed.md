title: 如何调试什么时候SaveFileDialog会被Dispose
date: 2015-03-25 19:37:30
categories:
tags: [CSharp, Debug]
description: 本文介绍了如何通过加函数断点的方式调试什么时候SaveFileDialog会被Dispose，然后通过调试结果说明我们在使用完SaveFileDialog后应该显示的Dispose。
---

今天又看到一个Coverity报出的问题，说`SaveFileDialog`在使用完之后没有`Dispose`。想起了之前写的一篇博文[你知道Form.Show()和Form.ShowDialog()的区别吗？](/2015/01/12/do-we-need-dispose-after-show-and-showdialog/)。那篇文章总结起来就是说，调用`Form.Show()`不需要显示的`Dispose`，但是调用`Form.ShowDialog()`需要显示的`Dispose`。那么回到今天的这个问题，`SaveFileDialog`在使用完之后需要显示的`Dispose`吗？如果看看MSDN的[示例代码](https://msdn.microsoft.com/en-us/library/system.windows.forms.savefiledialog%28v=vs.110%29.aspx)，其实它是没有显示`Dispose`的。那究竟需不需要呢？

还是写段程序来检查一下吧，看看什么时候`SaveFileDialog`会被`Dispose`。写一个WinForm，然后加2个按钮，它们对应的响应事件如下：

```c#
private void button1_Click(object sender, EventArgs e)
{
	SaveFileDialog s = new SaveFileDialog();
	if (DialogResult.OK == s.ShowDialog())
		MessageBox.Show("OK");
	else
	{
		MessageBox.Show("Not OK");
	}
	//s.Dispose();
}

private void button2_Click(object sender, EventArgs e)
{
	GC.Collect();
	GC.WaitForPendingFinalizers();
	GC.Collect();
}
```

然后在Visual Studio中加一个函数断点`System.ComponentModel.Component.Dispose`，因为`SaveFileDialog`继承自`Component`，所以当`SaveFileDialog`被`Dispose`时，这个断点应该会被触发。

运行这个程序，点第一个按钮，点完之后发现断点没有被触发。这个时候关掉程序，发现断点被触发了，这个时候要注意看是不是`SaveFileDialog`，因为好多都是从`Component`继承下来的，也可以在断点上加一个条件，这样就比较容易观察了。这说明`SaveFileDialog`在使用完后没有直接被`Dispose`，一直呆到了程序结束。

再次运行这个程序，点第一个按钮，然后点第二个按钮，这是发现断点被触发了，说明GC会帮我们清理这个没用到的局部变量。

如果把上面代码中的`Dispose`去掉注释，发现在第一个按钮按完之后就会被`Dispose`了。

综上，对于`SaveFileDialog`，我们还是应该在使用完后就显示的`Dispose`。

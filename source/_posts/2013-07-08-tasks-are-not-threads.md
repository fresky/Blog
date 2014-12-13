---
layout: post
title: "TPL中的task并不是thread"
date: 2013-07-08
comments: true
tags: CSharp
---
<p><a href="http://blogs.msdn.com/b/benwilli/archive/2013/04/24/tasks-are-not-threads.aspx">Tasks are not Threads - The Brain Dump</a>用了一个非常简单直观的例子说明了task和thread并不是一回事（尽管你调用Task.Run一般会在线程池上启一个线程帮你做些事情）。</p>  <p>假设有个UI，我们有2个按钮，开始和结束。代码如下：</p>  

```c#
private async void bttnStart_Click(object sender, RoutedEventArgs e)
{
    MessageBox.Show("Looking for the time");
    DateTime now = await GetCurrentTimeAsync();
    MessageBox.Show("It is now" + now.ToString());
}

private void bttnEnd_Click(object sender, RoutedEventArgs e)
{
    if (_tsc != null)
    {
        _tsc.SetResult(DateTime.Now);
    }
}

TaskCompletionSource<DateTime> _tsc;

private Task<DateTime> GetCurrentTimeAsync()
{
    _tsc = new TaskCompletionSource<DateTime>();
    return _tsc.Task;
}
```

<p>从上面的代码我们可以看到，这个task的开始于用户点击“开始”按钮，介绍于用户点击“结束”按钮。task其实在等用户的操作，同样的，task也可以在等网络，内存或者任何别的东西，不一定是线程。</p>
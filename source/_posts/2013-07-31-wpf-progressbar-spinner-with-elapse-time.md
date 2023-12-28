---
layout: post
title: "WPF的进度条progressbar，运行时间elapse time和等待spinner的实现"
date: 2013-07-31
comments: true
tags: Programming
---
<p>今天用.NET 4.5中的TPL的特性做了个小例子，实现了WPF的进度条progressbar，运行时间elapse time和等待spinner。</p>  <p>先上图吧。</p>  <p><img src="https://raw.github.com/fresky/WPFWaiterExample/master/screenshot.png" /></p>  <p>&#160;</p>  <p>这个例子包含4个实现，分别是同步版本（Sync），异步版本（Async），并发版本（Parallel）和通过数据绑定实现的并发版本（Parallel with Data Binding）。代码放在了<a href="https://github.com/fresky/WPFWaiterExample">Github</a>上。其中Spinner的实现来源于stackoverflow上Drew Noakes提供的<a href="http://stackoverflow.com/a/1492141/304115">代码</a>。</p>  <h3>1. 同步版本（Sync）</h3>  <p>这个版本中进度条、运行时间都不能更新，而且用户不能取消，因为所有的工作都是在UI线程中做的，整个UI被阻塞了。示例代码如下：</p>  

```csharp
internal override void Start()
{
	startWaiting();
	for (int i = 1; i <= Job.JobNumber; i++)
	{
		Job.TimeConsumingJob();
		m_FinishedJob++;
		m_Progressbar.Value = m_FinishedJob;
	}
	stopWaiting();
}
```

<h3>2. 异步版本（Async）</h3>

<p>使用C#的`await`和`async`关键字实现异步调用，这样进度条、运行时间都可以更新了，而且用户可以取消，因为UI没有被阻塞。示例代码如下：

```csharp
        internal override async void Start()
        {
            startWaiting();

            try
            {
                for (int i = 1; i <= Job.JobNumber; i++)
                {
                    await Task.Factory.StartNew(Job.TimeConsumingJob, m_CancellationTokenSource.Token);
                    m_FinishedJob++;
                    m_Progressbar.Value = m_FinishedJob;
                }
            }
            catch (OperationCanceledException)
            {
                m_CancellationTokenSource = new CancellationTokenSource();
            }
            
            stopWaiting();
        }
```

<h3><code><code></code></code>3. 并发版本（Parallel）</h3>

<p>把后台的工作都并发处理了，除了不阻塞UI之外处理速度得到了提高。示例代码如下：</p>

```csharp
        internal override async void Start()
        {
            startWaiting();

            List<Task> taskList = new List<Task>();
            for (int i = 1; i <= Job.JobNumber; i++)
            {
                taskList.Add(
                        Task.Factory.StartNew(Job.TimeConsumingJob).ContinueWith(t =>
                            {
                                m_FinishedJob++;
                                m_Progressbar.Value = m_FinishedJob;
                            },
                        m_CancellationTokenSource.Token,
                        TaskContinuationOptions.None,
                        TaskScheduler.FromCurrentSynchronizationContext())
                        );
            }

            try
            {
                await Task.WhenAll(taskList);
            }
            catch (OperationCanceledException)
            {
                m_CancellationTokenSource = new CancellationTokenSource();
            }

            stopWaiting();
        }
```

<h3>4. 通过数据绑定实现的并发版本（Parallel with Data Binding）</h3>

<p>一样是并发，但是用了Data Binding，没有直接操作UI控件。</p>
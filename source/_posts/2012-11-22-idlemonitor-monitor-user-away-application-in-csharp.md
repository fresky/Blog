---
layout: post
title: "（C#）IdleMonitor 实现监测用户是否离开应用程序"
date: 2012-11-22
comments: true
tags: CSharp
---
<p>写了一个C#的小程序，用来监测用户是否离开应用程序，就像MSN，QQ的离开功能一样。代码在<a href="https://github.com/fresky/IdleMonitor">github</a>上。简单接介绍一下：</p>  <p><strong>1. IdleMonitor abstract class</strong></p>  

```csharp
	public abstract class IdleMonitor
    {
        // ...
        public event EventHandler TimeoutEventHandler;

        protected IdleMonitor(TimeSpan timeout){...}
        public abstract void Start();
        public abstract void Stop();
        //...
    }
```

<p>&#160;</p>

<p>在WinForm和WPF的应用中，只需要通过IdleMonitorFactory得到一个IdleMonitor，然后调用Start()来开始监测，调用Stop()来结束监测。应用程序可以加event handler到TimeoutEventHandler上，这样当时间到了的时候就能被调到。</p>

<p><strong>2. IdleMonitor 的实现。</strong></p>

<p>实现了3中IdleMonitor。</p>

<p>（1）GetLastInputInfoIdleMonitor。</p>

<p>使用了<a href="http://msdn.microsoft.com/en-us/library/windows/desktop/ms646272%28v=vs.85%29.aspx">GetLastInputInfo</a> API。WinForm和WPF都能用。</p>

<p>（2）MessageFilterIdleMonitor。</p>

<p>使用了MessageFilter，过滤键盘和鼠标消息。所有的消息列表在<a href="http://wiki.winehq.org/List_Of_Windows_Messages">这里</a>。WinForm和WPF都能用。</p>

<p>（3）ComponentDispatcherIdleMonitor。</p>

<p>用到了<a href="http://msdn.microsoft.com/en-us/library/system.windows.threading.dispatcherhooks.operationposted.aspx">Operationposted</a> and <a href="http://msdn.microsoft.com/en-us/library/system.windows.interop.componentdispatcher.threadidle%28v=vs.110%29.aspx">ThreadIdle</a> 事件。只能在WPF中使用。</p>

<p>3. WinForm和WPF的使用例子。</p>

<p>combobox中是可选的3种IdleMonitor。</p>

{% limg IdleMonitor1.png %}

<p>打开IdleMonitor。</p>

{% limg IdleMonitor2.png %}

<p>时间到的时候会调用wpf中写好的event handler，抛如下对话框。</p>

{% limg IdleMonitor3.png %}

<p>在WinForm中的例子类似。</p>
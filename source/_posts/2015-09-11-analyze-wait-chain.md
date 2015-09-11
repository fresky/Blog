title: 使用Windows的分析等待链（analyze wait chain）来诊断没用响应的应用
date: 2015-09-11 21:09:18
categories:
tags: [Tool, Debug]
description:
---
我之前的博文[使用ProcDump在程序没有响应时自动收集dump](http://fresky.github.io/2015/06/03/use-procdump-to-detect-hung-window/)介绍过如何使用ProcDump自动收集无响应程序的dump。另外还有一个很有用的工具[WhatIsHang](http://www.nirsoft.net/utils/what_is_hang.html)也可以自动监测没有相应的程序，还能给出一个分析报告，包含调用栈Call Stack，栈上数据Stack Data等。

今天再看一个从Windows 7开始引入的这个分析等待链的功能，在Windows 7系统中打开Resource Monitor，在CPU页面，会列出所有的进程，右键点击某个进程，可以在右键菜单中看到分析等待链（analyze wait chain）。从Windows 8开始在任务管理器中可以直接右键进程，看到这个选项了。

下面是一个简单的示例，假设我们有如下程序：

```c#
static void Main(string[] args)
{
	Console.ReadLine();
}
```

运行之后针对这个进程点击分析等待链（analyze wait chain），我们可以看到如下结果。

{% limg analyzewaitchain.png %}

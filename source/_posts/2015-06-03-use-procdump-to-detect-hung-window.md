title: 使用ProcDump在程序没有响应时自动收集dump
date: 2015-06-03 17:39:05
categories:
tags: Debug
description:
---
我在[如何生成Dump文件](/2015/03/16/how-to-capture-dump-file/)中提到过可以使用[ProcDump](https://technet.microsoft.com/en-us/sysinternals/dd996900.aspx)根据CPU使用情况，异常情况，程序是否没有反应，性能计数等来收集dump。

今天举个具体的例子看看怎么用ProcDump来自动收集Hung的dump，ProcDump的`-h`参数会在window不响应window消息（就是我们通常在应用程序标题栏看到提示说没反应了）超过5秒钟时自动收集一个dump。

> -h	Write dump if process has a hung window (does not respond to window messages for at least 5 seconds).

先来一个简单的程序，一个Form，第一个按钮是进入无限循环。

```csharp
private void button1_Click(object sender, EventArgs e)
{
	int count = 0;
	while (true)
	{
		count++;
	}
	MessageBox.Show(count.ToString());
}
```

第二个按钮是触发一个死锁。

```csharp
private void button2_Click(object sender, EventArgs e)
{
	Thread t1 = new Thread(() =>
	{
		lock (a)
		{
			Thread.Sleep(100);
			lock (b)
			{
				MessageBox.Show("1");
			}
		}
	});

	Thread t2 = new Thread(() =>
	{
		lock (b)
		{
			Thread.Sleep(100);
			lock (a)
			{
				MessageBox.Show("2");
			}
		}
	});

	t1.Start();
	t2.Start();
	t1.Join();
	t2.Join();
}
```

运行程序，然后在命令行运行`procdump.exe -h form1.vshost.exe`。可以看到如下的输出。

```
ProcDump v7.1 - Writes process dump files
Copyright (C) 2009-2014 Mark Russinovich
Sysinternals - www.sysinternals.com
With contributions from Andrew Richards

Process:               Form1.vshost.exe (13088)
CPU threshold:         n/a
Performance counter:   n/a
Commit threshold:      n/a
Threshold seconds:     n/a
Hung window check:     Enabled
Log debug strings:     Disabled
Exception monitor:     Disabled
Exception filter:      *
Terminate monitor:     Disabled
Cloning type:          Disabled
Concurrent limit:      n/a
Avoid outage:          n/a
Number of dumps:       1
Dump folder:           D:\SysinternalsSuite\
Dump filename/mask:    PROCESSNAME_YYMMDD_HHMMSS


Press Ctrl-C to end monitoring without terminating the process.
```

这表明ProdDump开始监测了，点击第一个按钮，程序hung了。可以看到命令行如下的输出，ProcDump帮我们抓了一个dump。

```
[17:43:54] Hung Window:
[17:43:54] Dump 1 initiated: D:\SysinternalsSuite\Form1.vshost.exe_150603_174354.dmp
[17:43:55] Dump 1 complete: 11 MB written in 0.3 seconds
[17:43:55] Dump count reached.
```

如果点击第二个按钮，程序hung了，但是并没有触发ProcDump。所以ProcDump只能处理不响应的window消息的情况，对于死锁就无能为力了。

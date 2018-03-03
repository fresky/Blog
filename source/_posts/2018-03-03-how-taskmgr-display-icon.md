title: Windows的任务管理器怎么显示进程的图标
date: 2018-03-03 11:20:21
categories:
tags: Other
description:
---
[How does Task Manager choose the icon to show for a process?](https://blogs.msdn.microsoft.com/oldnewthing/20180301-00/?p=98135)介绍了Windows的任务管理器是如何显示进程的图标的。

* 如果进程有可见的窗口，那么就选择窗口的图标。  
* 如果进程没有可见的窗口，但是有右下角的提醒图标，那就选择这个提醒图标。  
* 如果进程没有可见的窗口，也没有右下角的提醒图标，那么任务管理器会用一个缺省的图标。  
* 针对一些特殊的进程，比如`svchost.exe`，有一些硬编码的特殊图标。
title: 如何生成Dump文件
date: 2015-03-16 00:52:11
categories: 
tags: Debug
description: 本文总结了在windows平台下可以生成dump的工具，包括Windbg, Visual Studio，任务管理器，Process Explorer，Windows的API，ProcDump，DebugDiag和WER。 
---
在windows平台部署完应用后，如果出现了程序崩溃的问题，我们首先需要的就是要生成一个dump文件，这样我们才能知道当时发生了什么，本文就简单总结一下如何生成Dump文件。

1. Windbg/ADPlus的`.dump`命令。如果要带heap信息的话，就需要用`/ma`参数。

2. Visual Studio的"Save Dump As ..."命令，在弹出的保存对话框中可以选择要不要保存heap信息。

3. 任务管理器，可以直接在对应的进程上点右键，"Create Dump File"。但是这个要注意的是如果你的系统是64位的，那么默认的任务管理就是64位的，这样就算你选中的是32位的程序，生成的dump也会使64位的。如果想生成32位的dump，需要找到32位的任务管理器。

4. 用[Process Explorer](http://technet.microsoft.com/en-us/sysinternals/bb896653.aspx)，和任务管理器差不多，在进程上点右键菜单，选择"Create Dump"，然后可以选择是mini dump还是full dump。

5. 直接使用Windows的API——[MiniDumpWriteDump](https://msdn.microsoft.com/en-us/library/ms680360.aspx)。

6. 使用[ProcDump](https://technet.microsoft.com/en-us/sysinternals/dd996900.aspx)，这个工具可以根据CPU使用情况，异常情况，程序是否没有反应，性能计数等来收集dump。

7. 使用[DebugDiag](http://www.microsoft.com/en-us/download/details.aspx?id=26798)，它可以监控内存泄露，另外它自动分析dump的结果也很友好。

8. Windows Error Reporting（WER），就是大家看到windows蓝屏后自动收集的那种dump，这个需要到[微软网站](https://msdn.microsoft.com/en-us/library/windows/hardware/dn641144.aspx)上注册。

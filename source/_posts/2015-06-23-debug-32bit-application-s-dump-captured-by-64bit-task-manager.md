title: "用wow64exts调试64位任务管理器抓取的32位程序的dump"
date: 2015-06-23 13:52:54
categories:
tags: Debug
description:
---
我在之前的博客[如何生成Dump文件](/2015/03/16/how-to-capture-dump-file/)中提到过，如果要用任务管理器收集32位应用程序dump的话，一定要确保用的是32位的任务管理器。假设用64位的任务管理器的话，用windbg打开，会发现类似如下的call stack。

```
0:000> ~*k

.  0  Id: 1b00.59d0 Suspend: 1 Teb: 00000000`fffdb000 Unfrozen
Child-SP          RetAddr           Call Site
00000000`0008e2e8 00000000`7443aedc wow64win!NtUserGetMessage+0xa
00000000`0008e2f0 00000000`7448d18f wow64win!whNtUserGetMessage+0x30
00000000`0008e350 00000000`74412776 wow64!Wow64SystemServiceEx+0xd7
00000000`0008ec10 00000000`77c301e4 wow64cpu!ServiceNoTurbo+0x2d
00000000`0008ec18 00000000`74480023 ntdll_77c20000!RtlUserThreadStart
00000000`0008ec20 00000000`00000246 wow64!_imp_memcpy <PERF> (wow64+0x23)
00000000`0008ec28 00000000`0018fff0 0x246
00000000`0008ec30 00000000`0008ec60 0x18fff0
00000000`0008ec38 00000000`00390008 0x8ec60
00000000`0008ec40 00000000`00000000 0x390008

   1  Id: 1b00.6040 Suspend: 1 Teb: 00000000`fffd8000 Unfrozen
Child-SP          RetAddr           Call Site
00000000`003af0f8 00000000`7441283e wow64cpu!CpupSyscallStub+0x9
00000000`003af100 00000000`77c301e4 wow64cpu!WaitForMultipleObjects32+0x3b
00000000`003af108 00000000`00000023 ntdll_77c20000!RtlUserThreadStart
00000000`003af110 00000000`00000246 0x23
00000000`003af118 00000000`042ffff0 0x246
00000000`003af120 00000000`00000000 0x42ffff0
```

可以用[!wow64exts.sw](https://msdn.microsoft.com/en-us/library/windows/desktop/aa384163%28v=vs.85%29.aspx)命令切换到x86模式下，这样可以看到非托管的call stack，也可以看变量值等。

需要注意的是对于托管代码，还是没用的，sos不能正确加载，所以看不到托管的调用栈信息。

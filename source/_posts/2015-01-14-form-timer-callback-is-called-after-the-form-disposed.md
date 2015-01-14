title: 为什么Form.Timer的event handler在Form被Dispose之后还是被调到了？
date: 2015-01-14 19:26:23
categories:
tags: CSharp
description: Form的Timer在Form Dispose时一定要Dispose，而且不能假设Timer的event handler在Timer Dispose后就一定不会调到了。
---

之前在我的博客[谁动了我的timer？C#的垃圾回收和调试](/2013/06/20/where-is-my-timer-csharp-gc/)中举了一个使用`Threading.Timer`局部变量导致可能被垃圾回收，从而它对应的event handler不能被调用的问题。今天再分享一个Form已经被Dispose了，但是它的`Form.Timer`的event handler还是被调到的问题。C#中各种`Timer`的区别，可以参见博客[C#中5种timer的比较](/2013/07/09/compare-of-5-timers-in-csharp/)。

####首先应该在Form被Dispose时也Dispose它的Timer
如果是通过Designer拽进来的，那么它在创建时会写成：
```c#
this.timer1 = new System.Windows.Forms.Timer(this.components);
```

这样Form在Dispose时会Dispose这个Timer，没有问题。但是如果是我们自己手写的，写成了
```c#
this.timer1 = new System.Windows.Forms.Timer();
```
那在Form关闭Dispose时，Timer并不会被自动的Dispose，这样就会导致Timer的event handler还能被调到。

####其次，就算Despose了，也不能保证它的event handler不会被调到

在我遇到的这个问题里，可以看到Form已经被Dispose了，这个可以通过windbg的`!do`命令来看Form的成员变量`state`，我的dump里`!do`的部分结果如下：

```
      MT    Field   Offset                 Type VT     Attr    Value Name
...
6d5f3aa4  40001c0       4c         System.Int32  1 instance 17434636 state
...
```

翻开C#的源代码可以看到state的定义如下：
```c#
internal const int STATE_CREATED                = 0x00000001;
internal const int STATE_VISIBLE                = 0x00000002;
internal const int STATE_ENABLED                = 0x00000004;
internal const int STATE_TABSTOP                = 0x00000008;
internal const int STATE_RECREATE               = 0x00000010;
internal const int STATE_MODAL                  = 0x00000020;
internal const int STATE_ALLOWDROP              = 0x00000040;
internal const int STATE_DROPTARGET             = 0x00000080;
internal const int STATE_NOZORDER               = 0x00000100;
internal const int STATE_LAYOUTDEFERRED         = 0x00000200;
internal const int STATE_USEWAITCURSOR          = 0x00000400;
internal const int STATE_DISPOSED               = 0x00000800;
internal const int STATE_DISPOSING              = 0x00001000;
internal const int STATE_MOUSEENTERPENDING      = 0x00002000;
internal const int STATE_TRACKINGMOUSEEVENT     = 0x00004000;
internal const int STATE_THREADMARSHALLPENDING  = 0x00008000;
internal const int STATE_SIZELOCKEDBYOS         = 0x00010000;
internal const int STATE_CAUSESVALIDATION       = 0x00020000;
internal const int STATE_CREATINGHANDLE         = 0x00040000;
internal const int STATE_TOPLEVEL               = 0x00080000;
internal const int STATE_ISACCESSIBLE           = 0x00100000;
internal const int STATE_OWNCTLBRUSH            = 0x00200000;
internal const int STATE_EXCEPTIONWHILEPAINTING = 0x00400000;
internal const int STATE_LAYOUTISDIRTY          = 0x00800000;
internal const int STATE_CHECKEDHOST            = 0x01000000;
internal const int STATE_HOSTEDINDIALOG         = 0x02000000;
internal const int STATE_DOUBLECLICKFIRED       = 0x04000000;
internal const int STATE_MOUSEPRESSED           = 0x08000000;
internal const int STATE_VALIDATIONCANCELLED    = 0x10000000;
internal const int STATE_PARENTRECREATING       = 0x20000000;
internal const int STATE_MIRRORED               = 0x40000000;
```

把dump里的state和表明Disposed的2048`&`一下，就能看出来这个Form已经被Dispose掉了。然后`!do`这个Form的Timer，有如下结果：

```
0:000> !do 1b0ef780 
Name:        System.Windows.Forms.Timer
MethodTable: 6a21234c
EEClass:     69ff114c
Size:        48(0x30) bytes
File:        C:\windows\Microsoft.Net\assembly\GAC_MSIL\System.Windows.Forms\v4.0_4.0.0.0__b77a5c561934e089\System.Windows.Forms.dll
Fields:
      MT    Field   Offset                 Type VT     Attr    Value Name
6d5f25e8  4000199        4        System.Object  0 instance 00000000 __identity
6b99afd0  40002d2        8 ...ponentModel.ISite  0 instance 00000000 site
6b99921c  40002d3        c ....EventHandlerList  0 instance 00000000 events
6d5f25e8  40002d1       e0        System.Object  0   static 021719d8 EventDisposed
6d5f3aa4  4001f87       20         System.Int32  1 instance    35000 interval
6d5e8138  4001f88       24       System.Boolean  1 instance        0 enabled
6d5e96c8  4001f89       10  System.EventHandler  0 instance 1b4dbfb0 onTimer
6d5f0be0  4001f8a       28 ...Services.GCHandle  1 instance 1b0ef7a8 timerRoot
6a2116bc  4001f8b       14 ...TimerNativeWindow  0 instance 00000000 timerWindow
6d5f25e8  4001f8c       18        System.Object  0 instance 00000000 userData
6d5f25e8  4001f8d       1c        System.Object  0 instance 1b0ef7b0 syncObj
```

其中的`timerWindow`是null，从C#的Timer源代码中可以看到这表明这个Timer也被Dispose了。

```c#
protected override void Dispose(bool disposing) {
	if (disposing) {                
		if (timerWindow != null) {
			timerWindow.StopTimer();                    
		}

		Enabled = false;
	}
	timerWindow = null;
	base.Dispose(disposing);
}
```

所以不能假设Timer的event handler在Timer Dispose后就一定不会调到了。
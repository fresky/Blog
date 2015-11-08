---
layout: post
title: "C#如何阻止计算机进入屏保或者省电模式"
date: 2013-04-18
comments: true
tags: CSharp
---
<p>有时候我们希望阻止计算机进入屏保或者省电模式，比如计算机进入了屏保或者省电模式后可能会导致automation test失败，或者我们就是想在自己出去玩的时候让屏幕一直亮着让老板以为自己刚出去：）</p>
<p>那这篇文章介绍一下如何用C#程序阻止计算机进入屏保或者省电模式。</p>
<p>要实现这个功能可以使用下面两个API。</p>
<h3>1. <a href="http://msdn.microsoft.com/en-us/library/windows/desktop/aa373208%28v=vs.85%29.aspx">SetThreadExecutionState</a></h3>
<p>通过这个API应用程序可以通知系统它正在运行,从而阻止计算机进入屏保.</p>
<p>实例代码如下:</p>

```csharp
using System;
using System.Runtime.InteropServices;

namespace NoSleepMonitor
{
    static internal class NoSleepWay1
    {
       [Flags()]
        public enum EXECUTION_STATE : uint //Determine Monitor State
        {
            ES_AWAYMODE_REQUIRED = 0x00000040,
            ES_CONTINUOUS = 0x80000000,
            ES_DISPLAY_REQUIRED = 0x00000002,
            ES_SYSTEM_REQUIRED = 0x00000001
            // Legacy flag, should not be used.
            // ES_USER_PRESENT = 0x00000004
        }

        [DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)]
        private static extern EXECUTION_STATE SetThreadExecutionState(EXECUTION_STATE esFlags);

        public static void Sleep()
        {
            SetThreadExecutionState(EXECUTION_STATE.ES_CONTINUOUS);
        }

        public static void NoSleep()
        {
            SetThreadExecutionState(EXECUTION_STATE.ES_DISPLAY_REQUIRED | EXECUTION_STATE.ES_CONTINUOUS);
        }
    }
}
```

<h3>2.<a href="http://msdn.microsoft.com/en-us/library/windows/desktop/ms724947%28v=vs.85%29.aspx">&nbsp;</a><a href="http://msdn.microsoft.com/en-us/library/windows/desktop/ms724947%28v=vs.85%29.aspx">SystemParametersInfo</a></h3>
<p>通过这个API应用程序可以设置系统参数,其中可以设置屏保的参数,从而取消屏保.</p>
<p>示例代码如下:</p>

```csharp
using System.Runtime.InteropServices;


namespace NoSleepMonitor
{
    public class NoSleepWay2
    {
        [DllImport("user32", EntryPoint = "SystemParametersInfo", CharSet = CharSet.Auto, SetLastError = true)]
        private static extern int SystemParametersInfo(int uAction, int uParam, string lpvParam, int fuWinIni);

        private const int SPI_SETSCREENSAVEACTIVE = 0x0011;

        public static void NoSleep()
        {
            SystemParametersInfo(SPI_SETSCREENSAVEACTIVE, 0, "0", 0);
        }
        public static void Sleep()
        {
            SystemParametersInfo(SPI_SETSCREENSAVEACTIVE, 1, "0", 0);
        }
    }
}
```
<p>&nbsp;</p>
<p>我在<a href="https://github.com/fresky/NoSleepMonitor">github</a>上放了一个wpf的小程序,包含了上述代码.</p>
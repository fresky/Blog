---
layout: post
title: "C#判断一个进程是不是64位的"
date: 2012-11-27
comments: true
tags: CSharp
---
<p>C#中的Environment 类有 <a href="http://msdn.microsoft.com/en-us/library/system.environment.is64bitoperatingsystem%28VS.100%29.aspx">Is64BitOperatingSystem</a> 属性，可以判断当前操作系统是不是64bit的。还有<a href="http://msdn.microsoft.com/en-us/library/system.environment.is64bitprocess%28VS.100%29.aspx">Is64BitProcess</a> 属性，可以判断当前进程是不是64bit的。</p>  <p>但是如果要判断别的进程是不是64bit的，就需要用windows API，<a href="http://msdn.microsoft.com/en-us/library/ms684139%28v=vs.85%29.aspx"><code>IsWow64Process</code></a>。</p>  

```c#
static class ProcessDetector
    {
        public static bool IsWin64(Process process)
        {
            if (Environment.Is64BitOperatingSystem)
            {
                IntPtr processHandle;
                bool retVal;

                try
                {
                    processHandle = Process.GetProcessById(process.Id).Handle;
                }
                catch
                {
                    return false;
                }
                return Win32API.IsWow64Process(processHandle, out retVal) && retVal;
            }

            return false;
        }
    }

    internal static class Win32API
    {
        [DllImport("kernel32.dll", SetLastError = true, CallingConvention = CallingConvention.Winapi)]
        [return: MarshalAs(UnmanagedType.Bool)]
        internal static extern bool IsWow64Process([In] IntPtr process, [Out] out bool wow64Process);
    }
```
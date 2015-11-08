---
layout: post
title: "c#中判断一个process是32bit还是64bit"
date: 2012-10-09
comments: true
tags: CSharp
---

下面的代码能判断一个process是32bit还是64bit.

```csharp
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
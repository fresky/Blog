title: Windows下如何检测用户修改了系统时间并且把系统时间改回来
date: 2015-09-16 20:04:36
categories:
tags: Programming
description:
---
有些时候我们不希望用户在使用我们的软件时修改系统时间，那么怎么检测用户是否修改系统时间呢？Windows会在系统时间修改时发送[WM_TIMECHANGE](https://msdn.microsoft.com/en-us/library/windows/desktop/ms725498%28v=vs.85%29.aspx)消息，所以可以在C++的WindowProc函数中处理这个消息。如果是C#，可以直接用[SystemEvents.TimeChanged](https://msdn.microsoft.com/en-us/library/microsoft.win32.systemevents.timechanged.aspx)事件。这个事件有个[bug](https://connect.microsoft.com/VisualStudio/feedback/details/776003/systemevent-timechanged-is-fired-twice)，就是每次都会被触发两次。

如果还想做的更智能一点，想把时间再改回去，就需要知道用户修改时间前的时间是什么。一个可行的办法是在程序开始运行时记一个开始时间，然后用[Stopwatch](https://msdn.microsoft.com/en-us/library/system.diagnostics.stopwatch%28v=vs.110%29.aspx)这个类来掐表算算过去了多长时间。给Windows设置时间需要用到[SetSystemTime](https://msdn.microsoft.com/en-us/library/windows/desktop/ms724942%28v=vs.85%29.aspx)这个API。

下面是示例代码：

```csharp
public class Program
{
	private static Stopwatch s_Stopwatch;
	private static DateTime s_Start;
	private static int s_Count = 0;

	[DllImport("kernel32.dll")]
	private extern static uint SetSystemTime(ref SYSTEMTIME lpSystemTime);


	private struct SYSTEMTIME
	{
		public ushort wYear;
		public ushort wMonth;
		public ushort wDayOfWeek;
		public ushort wDay;
		public ushort wHour;
		public ushort wMinute;
		public ushort wSecond;
		public ushort wMilliseconds;
	}


	static void Main(string[] args)
	{
		SystemEvents.TimeChanged += (sender, eventArgs) =>
		{
			if (s_Count == 1)
			{
				s_Count = 0;
				return;
			}
			var realTime = s_Start + s_Stopwatch.Elapsed;
			var newTime = DateTime.Now;


			Console.WriteLine(newTime);
			Console.WriteLine("Should be:");
			Console.WriteLine(realTime);

			var utcTime = realTime.ToUniversalTime();

			SYSTEMTIME systemtime = new SYSTEMTIME();

			systemtime.wYear = (ushort)utcTime.Year;
			systemtime.wMonth = (ushort)utcTime.Month;
			systemtime.wDay = (ushort)utcTime.Day;
			systemtime.wHour = (ushort)utcTime.Hour;
			systemtime.wMinute = (ushort)utcTime.Minute;
			systemtime.wSecond = (ushort)utcTime.Second;
			systemtime.wMilliseconds = (ushort)utcTime.Millisecond;

			SetSystemTime(ref systemtime);
			Console.WriteLine("Change back!");
			Console.WriteLine();

			s_Count++;
		};


		s_Stopwatch = new Stopwatch();
		s_Start = DateTime.Now;
		s_Stopwatch.Start();    
		
		Console.ReadLine();
		s_Stopwatch.Stop();
	}
}
```

下面是一个运行结果，可以看到时间改了两次，因为我们的程序改了一次。注意上面代码中的`s_Count`用来绕过上面提到的触发两次event的bug，如果没有这个话，这段代码会死循环下去。。。因为我们一直再改时间。当然也可以做一个时间的比较，小于某个值我们就认为一样，就不修改了。

```
9/15/2015 8:01:32 PM
Should be:
9/16/2015 8:01:32 PM
Change back!

9/16/2015 8:01:32 PM
Should be:
9/16/2015 8:01:32 PM
Change back!
```

两外一个要注意的是如果程序是Windows服务的话，需要起一个隐藏的窗口来处理消息，不然`TimeChanged`收不到。

如果想直接禁止用户修改时间，可以参见这篇文章[How to Allow or Prevent Specific Users and Groups from Changing the Date and Time in Windows](http://www.sevenforums.com/tutorials/113557-date-time-allow-prevent-users-groups-changing.html)。里面提供了两个方法：

1. 修改Local Security Policy中的Change the system time。  
1. 通过`ntrights运行ntrights -U "Administrators" -R SeSystemtimePrivilege`来禁止修改时间，运行`ntrights -U "Administrators" -R SeSystemtimePrivilege`来允许修改时间。
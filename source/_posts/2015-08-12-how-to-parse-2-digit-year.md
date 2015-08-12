title: C#如何转换2位数字表示的年
date: 2015-08-12 19:15:09
categories:
tags: CSharp
description: C#根据Calendar.TwoDigitYearMax来做2位数字年份的转换，默认值是2029，所以30就是1930，29就是2029。
---

C#的[DateTime.Parse](https://msdn.microsoft.com/en-us/library/system.datetime.parse%28v=vs.110%29.aspx)可以把一个字符串转换成DateTime。但是如果年份只有2位数字，它会怎么转换呢？来看看下面的代码：

```c#
DateTime d = DateTime.Parse("01/13/30");
Console.WriteLine(d);
```

输入结果是：
```
1/13/1930 12:00:00 AM
```

如果换成这样：
```c#
DateTime d = DateTime.Parse("01/13/29");
Console.WriteLine(d);
```

输入结果是：
```
1/13/2029 12:00:00 AM
```

怎么回事呢？貌似29是一个分水岭，29之前的是20XX，29之后的是19XX。

再去仔细看看MSDN吧，上面关于2位数字的年份怎么转换是这么说的：

> A string with a date but no time component. If the time component is absent, the method assumes 12:00 midnight. If the date component has a two-digit year, it is converted to a year based on the Calendar.TwoDigitYearMax of the current culture's current calendar or the specified culture's current calendar (if you use an overload with a non-null provider argument). 

就是说关于2位数字的年份转换，需要参考`Calendar.TwoDigitYearMax`，它是两位数字年份转换的最大值。那这个默认是2029是怎么来的呢？在[Calendar.TwoDigitYearMax](https://msdn.microsoft.com/en-us/library/system.globalization.calendar.twodigityearmax%28v=vs.110%29.aspx)有如下的描述：

> The initial value of this property is derived from the settings in the regional and language options portion of Control Panel. However, that information can change during the life of the AppDomain. The Calendar class does not detect changes in the system settings automatically. If the calendar is not supported in the regional and language options, the initial value of this property is the default value defined by the Calendar class.

打开我的控制面板，发现了如下的设置。

{% limg 2digityear.png %}


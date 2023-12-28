---
layout: post
title: "C# string.indexof（string)返回1！！！"
date: 2012-07-25
comments: true
tags: Programming
---
下面的代码第一次indexof返回0，第二次indexof返回1。

```csharp
const string softHyphenPlusHyphen = "\xAD\x2D";
Console.WriteLine("softHyphenPlusHyphen.IndexOf(softHyphenPlusHyphen,
StringComparison.Ordinal): " +
softHyphenPlusHyphen.IndexOf(softHyphenPlusHyphen,
StringComparison.Ordinal));
Console.WriteLine("softHyphenPlusHyphen.IndexOf(softHyphenPlusHyphen): " + softHyphenPlusHyphen.IndexOf(softHyphenPlusHyphen));
```

[Best Practices for Using Strings in the .NET Framework](http://msdn.microsoft.com/en-us//library/dd465121%28v=vs.110%29.aspx)讲了如何在.net中正确的使用字符串。摘要如下：


When you develop with the .NET Framework, follow these simple recommendations when you use strings:

> 1.     Use overloads that explicitly specify the string comparison rules for string operations. Typically, this involves calling a method overload that has a parameter of type StringComparison.
> 1.     Use StringComparison.Ordinal or StringComparison.OrdinalIgnoreCase for comparisons as your safe default for culture-agnostic string matching.
> 1.     Use comparisons with StringComparison.Ordinal or StringComparison.OrdinalIgnoreCase for better performance.
> 1.     Use string operations that are based on StringComparison.CurrentCulture when you display output to the user.
> 1.     Use the non-linguistic StringComparison.Ordinal or StringComparison.OrdinalIgnoreCase values instead of string operations based on CultureInfo.InvariantCulture when the comparison is linguistically irrelevant (symbolic, for example).
> 1.     Use the String.ToUpperInvariant method instead of the String.ToLowerInvariant method when you normalize strings for comparison.
> 1.     Use an overload of the String.Equals method to test whether two strings are equal. 
> 1.     Use the String.Compare and String.CompareTo methods to sort strings, not to check for equality.
> 1.     Use culture-sensitive formatting to display non-string data, such as numbers and dates, in a user interface. Use formatting with the invariant culture to persist non-string data in string form.
> 1. 	   Avoid the following practices when you use strings:
> 1.     Do not use overloads that do not explicitly or implicitly specify the string comparison rules for string operations. 
> 1.     Do not use string operations based on StringComparison.InvariantCulture in most cases. One of the few exceptions is when you are persisting linguistically meaningful but culturally agnostic data. 
> 1.     Do not use an overload of the String.Compare or CompareTo method and test for a return value of zero to determine whether two strings are equal.
> 1.     Do not use culture-sensitive formatting to persist numeric data or date and time data in string form.

下表是如何选择StringComparison：

{% limg StringComparison.png %}

---
layout: post
title: "WPF中radiobutton 的 data binding方法"
date: 2012-08-06
comments: true
tags: Programming
---
WPF中的radiobox通过data binding绑定到一个bool属性后，如下所示，尽管UI可以正确的显示，但是data binding的属性不能正确的更新。比如user点了No之后属性UserChoice还是True。

```csharp
<RadioButton Content="Yes" IsChecked="{Binding UserChoice}"/>
<RadioButton Content="No" />
```

需要用如下的方式：
```csharp
<RadioButton Content="Yes" IsChecked="{Binding UserChoice}"/>
<RadioButton Content="No" IsChecked="{Binding UserChoice, Converter={StaticResource radioConverter}}"/>
```

`radioConverter`如下：

```csharp
public class RadioButtonConverter : IValueConverter
{
	public object Convert(object value, Type targetType, object parameter, System.Globalization.CultureInfo culture)
	{
		if (value is bool)
		{
			return !(bool)value;
		}
		return value;
	}

	public object ConvertBack(object value, Type targetType, object parameter, System.Globalization.CultureInfo culture)
	{
		if (value is bool)
		{
			return !(bool)value;
		}
		return value;
	}
}
```
这样就能正确更新了。
title: "你真的知道C#的TryParse吗？"
date: 2014-12-18 19:00:03
categories:
tags: CSharp
description: 本文从解决一个C#Double.TryParse的bug谈起，介绍了TryParse的缺省参数NumberStyles和IFormatProvider。
---
前段时间遇到一个问题，在一个UI上需要显示两个值的差值，所以在代码中使用了`Convert.ToDouble`，这个相当于`Double.Parse(String)`。一直工作的很好，直到有一天发现有应用给的这两个输入值是“1, 2”之类的字符串，并不是一个double数值了，于是程序就Crash了。

所以改成了`Double.TryParse`，这样如果失败就放弃，不计算差值。嗯，问题解决，不会crash了。接着就加了几个测试来覆盖这段逻辑，但是在测试过程中，发现如果输入是“1,2”，这个TryParse是返回true的，生成的double数值是12。这就有趣了，第一个猜测就是`Double.TryParse`把逗号当成了千位数的分隔符，直接去掉了。

翻翻MSDN找到了有两个重载的，一个是`Double.TryParse(String, Double)`，一个是`Double.TryParse(String, NumberStyles, IFormatProvider, Double)`。`Double.TryParse(String, Double)`中使用的NumberStyles是`NumberStyles.Float`和` NumberStyles.AllowThousands`。其中` NumberStyles.AllowThousands`就是表明支持用逗号作为千位数的分隔符。所以如果真的是需要处理浮点数，应该用如下的格式：

```csharp
double d;
bool b = double.TryParse(s, NumberStyles.Number, CultureInfo.InvariantCulture, out d);
```

从[NumberStyles Enumeration](http://msdn.microsoft.com/en-us/library/system.globalization.numberstyles.aspx)可以找到所有可能的NumberStyles。

在这个页面前面列出的允许的数字类型，包括  
AllowCurrencySymbol，  
AllowDecimalPoint，  
AllowExponent，  
AllowHexSpecifier，  
AllowLeadingSign，  
AllowLeadingWhite，  
AllowParentheses，  
AllowThousands，  
AllowTrailingSign，  
AllowTrailingWhite。

后面是一些常用的组合：

|组合名|描述|
|---|---|
|Any|除AllowHexSpecifier之外所有的格式|
|Currency|除AllowExponent 和 AllowHexSpecifier之外所有的格式|
|Float|AllowLeadingWhite, AllowTrailingWhite, AllowLeadingSign, AllowDecimalPoint, AllowExponent|
|HexNumber|AllowLeadingWhite, AllowTrailingWhite, AllowHexSpecifier|
|Integer|AllowLeadingWhite, AllowTrailingWhite, AllowLeadingSign。Int.TryParse的默认NumberStyles就是NumberStyles.Integer|
|None|只能包含数字|
|Number|AllowLeadingWhite, AllowTrailingWhite, AllowLeadingSign, AllowTrailingSign, AllowDecimalPoint, AllowThousands|

可以根据自己的需要选择合适的NumberStyles。当然如果想让AllowThousands真的只能表示千位分隔符，就必须祭出正则表达式了，比如`@"^-?\d{1,3}(,\d\d\d)*"`。

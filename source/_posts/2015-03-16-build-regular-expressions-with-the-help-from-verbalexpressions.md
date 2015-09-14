title: 用VerbalExpressions来帮助我们写正则表达式
date: 2015-03-16 19:42:00
categories:
tags: [Regex, Tool]
description: 使用VerbalExpressions可以在多种语言，包括C++和C#中用非常直观的方式来帮助我们写正则表达式。
---

正则表达式是一个非常强大的工具，如果能够正确的使用它会大大提高文本处理的效率。我在之前的博文[正则表达式中\d和[0-9]有什么区别](/2013/06/04/d-0-9-difference-in-regex/)和[[ -~] 所有的可打印字符](/2012/11/19/match-all-printable-character-in-regex/)中也提到过一些关于正则表达式的用法。但是正则表达式的语法就不是那么友好了，如果不常用的话经常忘记，需要查帮助。

现在给大家推荐一个叫做[VerbalExpressions](http://verbalexpressions.github.io/)的小工具，用它可以用类似自然语言的方式帮我们写出正则表达式来。他有很多语言的版本，[这是C#的](https://github.com/VerbalExpressions/CSharpVerbalExpressions)，[这是C++的](https://github.com/VerbalExpressions/CppVerbalExpressions)。

下面是一个C#的示例代码，很容易用吧：）

```c#
var verbEx = new VerbalExpressions()
			.StartOfLine()
			.Then("http")
			.Maybe("s")
			.Then("://")
			.Maybe("www.")
			.AnythingBut(" ")
			.EndOfLine();

// Create an example URL
var testMe = "https://www.google.com";

Console.WriteLine(verbEx.Test(testMe));
Console.WriteLine(verbEx);
```

输出结果为：
```
True
^(http)(s)?(://)(www\.)?([^\ ]*)$
```
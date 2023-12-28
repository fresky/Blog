---
layout: post
title: "ASP .NET readonly texbox 后台不能读取问题的解决办法"
date: 2010-10-09
comments: true
tags: Programming
---
最近在做一些ASP .NET和silverlight的项目，把遇到的一些问题记在这里吧：）<br /><br />如果需要有一个textbox只能接受特定格式的东西，一个做法是把这个textbox做成readonly的，然后通过别的方式，比如javascript动态的生成需要填充的东西，放进这个textbox里面。但是在ASP的程序中postback到后台后会发现拿不到这个textbox的值，原因在msdn里做了如下解释。<br /><br />

> The Text value of a TextBox control with the ReadOnly property set to true
> is sent to the server when a postback occurs, but the server does no
> processing for a read-only text box. This prevents a malicious user from
> changing a Text value that is read-only. The value of the Text property is preserved in the view state between postbacks unless modified by server-side code.

一个简单的解决办法是，在code behand中设置read only 属性，示例代码如下：
```csharp
textboxReadonly.Attributes.Add("readonly", "readonly");
```
这样的话就可以了。
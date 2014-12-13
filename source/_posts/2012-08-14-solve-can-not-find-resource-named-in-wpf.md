---
layout: post
title: "WPF中Cannot find resource named '{XXX}'的解决办法"
date: 2012-08-14
comments: true
tags: CSharp
---
今天遇到了一个wpf的bug，如果在App.xaml中没有指定StartupUri（而是想通过override OnStartup 来指定StartupUri），并且只有一个resource的话，会遇到Cannot find resource named '{XXX}'的错误。

解决这个问题的简单办法有两个：

1. 在App.xaml中指定一个`x:Name="App"`
2. 在Resource中添加一个没用的resource，`<Style x:Key="unused" />`
---
layout: post
title: "如何解决因为找不到Notepad++的安装路径而导致的不能更新CS-Script的问题"
date: 2014-04-21 17:05
comments: true
tags: [Tool,CSharp]
keywords: csscript,notepad++,installation,update
description: 如何通过添加注册表键值（HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\App Paths\notepad++.exe），解决因为找不到Notepad++的安装路径（cannot find notepad++ installation）而导致的不能更新CS-Script的问题。
---

我之前的博客[用CS-Script把Notepad++变身支持智能提示和运行代码的C#集成开发环境](/2014/03/31/make-your-notepadd-plus-plus-as-csharp-ide-with-intellisense-and-code-execution/)介绍过CS-Script这个超级好用的Notepad++插件，但是我在使用过程中发现了一个小问题。如果使用的Notepad++是Potable绿色版的，在更新CS-Script，或者安装msi时会报出如下的一个错误对话框。
>*cannot find notepad++ installation*
>*不能找到notepad++的安装路径*

{% limg csscriptinstallfail.png %}

这估计是插件作者在制作msi安装文件的时候从注册表里取Notepad++的安装路径，但是如果用Potable绿色版的话就没有相应的注册表键值。

解决这个问题的方法很简单，在注册表中加上如下的键值。
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\App Paths\notepad++.exe
它的Default Value写上notepad++.exe的全路径即可。

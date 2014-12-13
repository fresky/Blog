---
layout: post
title: "如何给Windows添加自动启动的程序"
date: 2014-05-05 18:13
comments: true
tags: Other
keywords: 
description: 
---
本文总结了一下在Windows中可以添加自动启动的程序的地方。起因是有些软件在安装完后会在Windows中加一些自动启动的程序，我非常的不能忍。非常不巧的是我经常需要重新安装的一个软件也有这个毛病，于是今天就想写个小脚本每次安装完后把自动启动的程序去掉。

- 文件夹  
```
C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
```

- 注册表（`HKEY_CURRENT_USER`和`HKEY_LOCAL_MACHINE`）  
```
Software\Microsoft\Windows\CurrentVersion\Run
Software\Microsoft\Windows\CurrentVersion\RunOnce
Software\Microsoft\Windows\CurrentVersion\RunOnceEx 
Software\Microsoft\Windows\CurrentVersion\RunServices
Software\Microsoft\Windows\CurrentVersion\RunServicesOnce
SOFTWARE\Microsoft\Shared Tools\MSConfig\startupreg
SOFTWARE\Microsoft\Shared Tools\MSConfig\startupfolder
```
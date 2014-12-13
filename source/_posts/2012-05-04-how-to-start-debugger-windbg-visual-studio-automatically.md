---
layout: post
title: "自动启动 Debugger（windbg，visual studio）"
date: 2012-05-04
comments: true
tags: Debug
---
<ol><li> 打开注册表，<b>HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Image File Execution Options</b>.<br /></li><li> 找到你想debug的程序，比如myapp.exe. （如果没有，就创建一个）。<br /></li><li> 右键myapp.exe文件夹, 添加一个String Value. 命名为: <b>debugger</b>, 值为: <b>vsjitdebugger.exe</b>.(用visual studio). 或者 <b>C:\Program Files (x86)\Debugging Tools for Windows (x86)\windbg.exe</b>.(using windbg)<br /></li></ol>
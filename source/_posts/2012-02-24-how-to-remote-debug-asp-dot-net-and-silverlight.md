---
layout: post
title: "远程调试ASP .NET和Silverlight"
date: 2012-02-24
comments: true
tags: [CSharp, Debug]
---
这几天有个网站突然出了问题，是ASP .NET + Silverlight。用下面的方法可以远程调试。<br /><br />ASP .NET远程调试。<br />1. 复制  [VSInstallFolder]\Common7\IDE\Remote Debugger\x86文件夹到服务器production server上。<br />2. 在服务器上运行msvsmon.exe。打开Tools-&gt;Options，选择Windows Authentication。<br />3. 在自己的机器上运行Visual Stuido，点Attach to process，然后在 Qualifier上输入服务器的IP。<br />4. 选择“Show processes from all users", attach到 w3wp.exe进程.<br />5. 如果你有symbols和source code，就能debug，打断点了。<br /><br />Silverlight是运行在客户端的， 不需要远程调试，直接打开Visual studio，attach到相应的进行就行了。对于IE来说一般是 iexplorer.exe，对于firefox，是plugin-container.exe。一般根据type，找有silverlight的就行了。<br />
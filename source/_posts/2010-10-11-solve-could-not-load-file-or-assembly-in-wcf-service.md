---
layout: post
title: "wcf web service遇到\"Could not load file or assembly App_Web...\"问题"
date: 2010-10-11
comments: true
tags: CSharp
---
在IIS中部署wcf webservice，遇到"Could not load file or assembly App_Web..."问题。<br /><br />原因是修改了wcf webservice的方法，需要把.svc文件覆盖原来的即可解决。详见下网页。<br /><br /><a href="http://forums.asp.net/t/1566085.aspx">Could not load file or assembly App_web_.... (WCF) - ASP.NET Forums</a><br /><p>It could be that whenever you change method signature (Add / remove  methods, change parameters. change method names etc.) it may happend.</p><p>You need, whenever you deploey the service class'es DLL, to copy the .svc file as well.</p><br /><blockquote></blockquote><br /><br />
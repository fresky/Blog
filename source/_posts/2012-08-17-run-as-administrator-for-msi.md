---
layout: post
title: "对msi的安装包 Run As Administrator"
date: 2012-08-17
comments: true
tags: Installer
---
刚才有个同事装msi一直不能成功，抛错：
```
error 1001 the directory name is invalid
```
发现是没有用管理员权限运行安装包，对于是exe的，可以直接右键“Run As Administrator”。

对于msi的，需要这样：
```
msiexec /a your.msi
```
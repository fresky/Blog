---
layout: post
title: "你能在windows上创建一个叫做AUX的文件夹吗？"
date: 2013-08-14
comments: true
tags: Other
---
<p>Windows的文件名不能有如下这些特殊符号，这个大家都比较熟悉了。</p><blockquote><li>&lt; (less than)</li><li>&gt; (greater than)</li><li>: (colon)</li><li>" (double quote)</li><li>/ (forward slash)</li><li>\ (backslash)</li><li>| (vertical bar or pipe)</li><li>? (question mark)</li><li>* (asterisk)</li></blockquote><p>不过其实还有一些文件名被设备名占用，是不能用的，比如CON（console），PRN（printer）等，如下：</p><p>CON, PRN, AUX,           NUL, COM1, COM2,           COM3, COM4, COM5,           COM6, COM7, COM8,           COM9, LPT1, LPT2,           LPT3, LPT4, LPT5,           LPT6, LPT7, LPT8, and           LPT9</p><p>更全的列表参见这里：<a href="http://msdn.microsoft.com/en-us/library/aa365247.aspx">Naming Files, Paths, and Namespaces (Windows)</a></p>
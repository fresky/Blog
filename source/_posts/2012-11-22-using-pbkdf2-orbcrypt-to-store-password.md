---
layout: post
title: "用PBKDF2 或BCrypt 来存储密码"
date: 2012-11-22
comments: true
tags: Design
---
<a href="http://blog.8thlight.com/adam-gooch/2012/11/04/taking-password-storage-up-a-notch.html">Taking Password Storage Up A Notch | 8th Light</a>中介绍了只是用Hash加盐的做法是不够安全的，应该用<a href="http://www.ietf.org/rfc/rfc2898.txt">PBKDF2</a> 或 <a href="http://www.openbsd.org/papers/bcrypt-paper.pdf">BCrypt</a>来做密码存储算法。PBKDF2用在了 iOS, Android, 和LastPass。BCrypt用于OpenBSD。<a href="http://msdn.microsoft.com/en-us/library/system.security.cryptography.rfc2898derivebytes.aspx">Rfc2898DeriveBytes</a>是C#的一个实现。<br /><blockquote><br /></blockquote>
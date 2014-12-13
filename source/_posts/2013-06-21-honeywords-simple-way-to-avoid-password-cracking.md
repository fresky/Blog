---
layout: post
title: "Honeywords项目——检查密码是否被破解的一种简单方法"
date: 2013-06-21
comments: true
tags: Tool
---
<p><a href="http://people.csail.mit.edu/rivest/honeywords/">Honeywords</a>项目使用一种简单的方法来改进hash后的密码的安全性&mdash;&mdash;为每个账户维护一个额外的honeywords（假密码）。如果有黑客拿到了密码的文件，然后试图用brute froce的方式破解密码的话，黑客不知道他找到的是真正的密码还是honeyword。对于服务器来说，添加一个辅助的honeychecer，就能在收到honeyword时发出警告，从而就知道了密码文件已经泄露了。</p><p>网站上放了作者的论文，FAQ和一个python实现的简单的honeyword生成器。</p>
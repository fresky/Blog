---
layout: post
title: "通过位操作简洁的写KB，MB，GB"
date: 2012-11-19
comments: true
tags: Programming
---
1 &lt;&lt; 10 = KB, 1 &lt;&lt; 20 = MB, 1 &lt;&lt; 30 = GB ... 所以如果想开一个2KB的buffer，可以写2&lt;&lt;10，比较简洁。<br />
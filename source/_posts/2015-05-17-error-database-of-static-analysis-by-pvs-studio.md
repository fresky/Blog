title: 从bug中学习怎么写代码
date: 2015-05-17 20:22:25
categories:
tags: [Programming,Tool]
description: 从PVS Studio的一个通过静态检查开源项目发现的bug数据库中学习怎么写代码。
---
我在之前的[来试试这个来自静态代码分析工具PVS Studio提供C++的小测验吧](/2014/12/22/cpp-quiz-from-pvs-studio/)中介绍过一个PVS Studio提供的关于C++的小测试，今天再介绍一个PVS Studio提供的bug数据库[Errors detected in Open Source projects by the PVS-Studio developers through static analysis ](http://www.viva64.com/en/examples/)。

这个bug数据库是用PVS Studio来扫描开源项目，然后记录发现的各种bug，可以看看，从bug中学习怎么写代码。

比如下面几个常见的错误。

1. V511，`sizeof()`运算符返回的是指针的大小，而不是数组的大小。  
1. V503，比较一个指针<0。
---
layout: post
title: "如何减少不能重现的Bug"
date: 2014-04-30 17:08
comments: true
tags: [Design,Testing]
keywords: 
description: 
---
[Anthony Vallone](http://www.anthonysapps.com/anthony)在Google的测试博客上的这篇文章[Minimizing Unreproducible Bugs](http://googletesting.blogspot.ca/2014/02/minimizing-unreproducible-bugs.html)中介绍了一些他的经验来减少不能重现的Bug，要点如下：

---
**避免，并且测试race condition，死锁，timing issue，memory corruption，非初始化的内存访问，内存泄漏和资源问题。**

##### 开发

- 简化同步逻辑。  
- 永远用相同的顺序来加锁。  
- 不要为了优化而创建很多细粒度的锁，除非你证明了他们是必须的。  
- 避免共享内存。  

##### 测试
- 有规律的进行压力测试  
- 测试超时的情况  
- 测试debug和release版本  
- 在限制资源的情况测试  
- 长时间的测试  
- 用动态分析工具检查  

---
##### 开发
- 在函数开头对输入进行检查，验证前提条件是否成立  

---
##### 开发
- defensive programming  
##### 测试
- fuzz testing  

---
**不要对用户隐藏所有的错误** 
##### 开发
- 只有在确定错误不会影响系统状态或者用户时才对用户隐藏  
- 任何对用户有影响的错误都应该汇报给用户，并且提供下一步的指示  

---
**测试错误处理**
##### 开发
- 永远测试错误处理代码  
- 检查所有错误类型对应的日志质量  

---
**检查重复的键值**
##### 开发
- 努力保证所有键值的唯一性  
- 当不能保证时，在使用前先检查是否已经使用过这个键值  
- 小心可能的race condition  

---
**测试并发的数据访问**
##### 测试
- 如果系统需要并发数据访问，就测试它  

---
**远离未定义的行为和不确定的数据访问**
##### 开发
- 理解使用的API或者操作在什么情况下会引发未定义的行为  
- 不要依赖数据结构的顺序（除非你保证他们的顺序）  

---

**在发生错误时记录详细的日志**
##### 开发
- 遵守好的[日志规范](http://googletesting.blogspot.com/2013/06/optimal-logging.html)  
- 如果日志在用户的机器上，提供一种简单的方式是你能够拿到日志  

##### 测试
- 保存测试产生的日志  
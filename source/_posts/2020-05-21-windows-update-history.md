title: Windows更新的格式变化
date: 2020-05-21 13:08:02
categories:
tags: Design
description:
---

2018年8月微软宣布了新的Windows更新设计**质量（Quality）更新**，我们来回顾一下Windows更新的格式变化来看看能给我们什么启发。

# 一个文件的更新
针对一个文件的更新，通常有两种方式，一个是**整个文件**，一个是**补丁**。

|| 整个文件|补丁|
|---|---|---|
|大小|大|小|
|适用性|可以更新任何版本|只能更新一个版本|

假设我们只有一个文件，并且这个文件的初始**主版本**是F0，之后每个月发布一个更新，分别是F1，F2，F3（这个月没有任何变化，所以F3==F2），F4和F5。那我们就会有如下的更新。

|更新|这个文件|补丁（主版本）|补丁（1月）|补丁（2月）|补丁（3月）|补丁（4月）|
|---|---|---|---|---|---|---|---|
|1月|F1|F0到F1||||
|2月|F2|F0到F2|F1到F2|||
|3月||||||
|4月|F4|F0到F4|F1到F4|F2到F4|||
|5月|F5|F0到F5|F1到F5|F2到F5||F4到F5|

下面我们扩展到多个文件的情况，假设我们系统的初始**主版本**是M0，之后每个月发布一个更新，分别是M1，M2，M2（这个月没有任何变化），M4和M5。

# 全量更新

全量更新最简单，它就是包含从**主版本**M0以来所有变化的文件，这就保证了无论你现在机器上装的是什么版本，你都能成功的更新到最新的版本。所以更新服务器上放的东西就如下所以。请注意在3月和2月的更新服务器上放的东西是一样的。Windows全量更新的大小大概是1G。

|全量更新| 服务器上的内容|
|---|---|
|1月|M1-M0|
|2月|M2-M0|
|3月|M2-M0|
|4月|M4-M0|
|5月|M5-M0|


# 增量（Delta）更新

这里增量（Delta）是文件级别的，不是字节级别的。就是说**增量**（Delta）和**补丁**没关系。增量更新是全量更新的一个缩减版本，就是包含了从**上一个版本**以来所有变化的文件。Windows增量更新的大小大概是300M到500M。这样只要用户的版本一直是保持最新的，就能用较小的更新大小更新到下一个版本了。

|增量更新| 服务器上的内容|
|---|---|
|1月|M1-M0|
|2月|M2-M1|
|3月|无|
|4月|M4-M2|
|5月|M5-M3|

# 快速（Express）更新

快速（Express）更新是在全量更新的基础上加上了从最开始到最新版本的**补丁**。Windows通常补丁的大小是150M到200M。这样大部分情况都能找到针对某个客户端的补丁，从而实现通过补丁的方式来更新，更少的网络传输。如果你装了不是每月补丁的东西（比如hotfix补丁）那么就需要回退到全量更新的方式来更新。

|快速更新|服务器上的内容|
|---|---|---|
|1月|M1，M1-M0补丁|
|2月|M2，M2-M0补丁，M2-M1补丁|
|3月|M2，M2-M0补丁，M2-M1补丁|
|4月|M4，M4-M0补丁，M4-M1补丁，M4-M2补丁|
|5月|M5，M5-M0补丁，M5-M1补丁，M5-M2补丁，M5-M4补丁|

# 质量（Quality）更新

质量（Quality）更新包含两个部分：
* 从初始版本到现在的补丁，我们叫它**正向补丁**
* 从现在版本到初始版本的补丁，也就是**反向补丁**

这两个补丁的大小大概是250M。对于客户端来说，每次更新，都会用上次更新中的那个**反向补丁**把系统回退到初始状态，然后再用这次的**正向补丁**来升级。

|质量更新| 服务器上的内容|
|---|---|
|1月|M1-M0补丁，M0-M1补丁|
|2月|M2-M0补丁，M0-M2补丁|
|3月|M2-M0补丁，M0-M2补丁|
|4月|M4-M0补丁，M0-M4补丁|
|5月|M5-M0补丁，M0-M5补丁|

现在质量（Quality）更新已经取代了之前的所有的更新方式，成为唯一的更新方式。

# 各种更新方式的比较

||全量更新|增量更新|快速更新|质量更新|
|---|---|---|---|---|
|更新成功前提|总能成功|只有当前系统装了上次更新才能更新成功|总能成功，因为有全量更新作为退路|总能成功|
|服务器端存储|大（大概1G）|全量的1/3到1/2（300M-500M）|体积比全量更新大，但是平均网络带宽只需要全量更新的1/7到1/5|全量的1/3|
|和服务器交互|少|较少|多，客户端需要量身定制下载什么|少|
|对缓存友好|是|是|不太友好|是|
|服务器端逻辑|简单|简单|复杂|简单|


# 参考阅读
本文参考了下面这些文章：

[Survey of Windows update formats: The Full update](https://devblogs.microsoft.com/oldnewthing/20200210-00/?p=103426)  
[Survey of Windows update formats: The Delta update](https://devblogs.microsoft.com/oldnewthing/20200211-00/?p=103430)  
[Survey of Windows update formats: The Express update](https://devblogs.microsoft.com/oldnewthing/20200212-00/?p=103434)  
[Survey of Windows update formats: The Quality update, which obsoletes all the others](https://devblogs.microsoft.com/oldnewthing/20200213-00/?p=103436)  
[Quality updates: Consequences for rogue-patched binaries](https://devblogs.microsoft.com/oldnewthing/20200214-00/?p=103438)  
[What’s next for Windows 10 and Windows Server quality updates](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/what-s-next-for-windows-10-and-windows-server-quality-updates/ba-p/229461)  
[Windows 10 quality updates explained and the end of delta updates](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/windows-10-quality-updates-explained-and-the-end-of-delta/ba-p/214426)  
---
layout: post
title: "如何在Windows中打开多个Windows Media Player"
date: 2014-08-11 11:54
comments: true
tags: [Programming,Debug]
keywords: Mutex,Semaphore,Event,Critical Section
description: 如何在Windows中打开多个Windows Media Player，本文介绍了Windows中的Mutex、Semaphore、Event和Critical Section的区别。
---

在很多情况下我们需要保证我们的应用只有一个实例在运行，要实现这个目的有很多种做法，比如  
1. 我们可以在启动程序时遍历现在运行的进程列表，来看看我们这个进程是不是已经在运行了，如果已经在运行了就直接退出。  
2. 我们可以在启动程序时在硬盘生成一个文件，下次启动时来检查这个文件。  
3. 等等。。。  

我们知道在windows中是不能打开多个Windows Media Player的，下面来看看Windows Media Player是怎么做的。

打开一个Windows Media Player，然后用[Process Explorer](http://technet.microsoft.com/en-US/sysinternals)来看看Windows Media Player这个进程下的句柄，如下图所示：

{% limg wmp_mutex.png %}

可以看到wmplayer.exe这个进程有一个Mutant，名字是Mcirosoft_WMP_70_CheckForOtherInstanceMutex，从名字也能猜到这个Mutex是干啥用的。原来Windows Media Player是使用Mutex来保证只有一个Windows Media Player的实例。

我们来验证一下，在这个句柄上点右键，然后点击“Close Handle"，把这个handle关掉。然后在运行wmplayer.exe，这个时候就可以看到我们可以打开第二个Windows Media Player了。

下面简单对比一下Windows中的Mutex、Semaphore、Event和Critical Section。

1. Mutext，也叫做Mutant。只允许一个线程进入，这个进入的线程被认为是Mutex的所有者。所有者可以重入。Mutex的所有者需要在操作完成后释放这个Mutex，如果没有释放就结束了，操作系统会释放这个Mutex，但是会设置成**Abandoned Mutex**。
2. Semaphore。维护了一个计数器，每当一个线程获取这个Semaphore，就会减一。当线程释放这个Semaphore，就会加一。**Semaphore不维护所有者信息**。
3. Event。维护一个布尔标志。分为自动和手动两种。
4. Critical Section。和Mutex类似，但是是**用户态**的对象，前面3个都是内核态的。只能用在进程内。线程退出前必须释放（LeaveCriticalSection），否则其他等待这个Critical Section的线程会永远等待下去。



顺便说一下Windows的API：WaitForSingleObject或者WaitForMultipleObjects中所谓的Signaled在不同的对象中指的是什么意思：

1. 进程（线程），Signaled指的是进程（线程）结束了。
2. Mutex，Signaled指的是Mutex是Free的。
3. Event，Signaled指的是Event flag is raised。
4. Semaphore，Signaled指的是Semaphore的count大于0.
5. 文件，Signaled指的是I/O操作结束了。
6. Timer，Signaled指的是过期了。

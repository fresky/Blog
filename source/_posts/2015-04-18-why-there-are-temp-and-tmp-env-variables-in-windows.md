title: 为什么在Windows有两个临时文件夹的环境变量Temp和Tmp？
date: 2015-04-18 17:58:43
categories:
tags: Other
description:
---
如果看看windows的环境变量，可以看到`Temp`和`Tmp`同时存在，为啥会有两个呢？我们应该使用哪个呢？昨天The Old New Thing的博文[Why are there both TMP and TEMP environment variables, and which one is right?](http://blogs.msdn.com/b/oldnewthing/archive/2015/04/17/10608077.aspx)讲了讲这段历史。

在MS-DOS之前的操作系统是没有环境变量的，尽管MS-DOS已经有了环境变量的概念，但是因为之前的程序员都没用过，所以刚开始大家都没有用。随着时间的发展，程序员们发现可以用环境变量来存储一些东西比如配置文件，所以不同的先行者们就是用了`TEMP`和`TMP`两个不同的环境变量来指明临时文件夹的位置。

MS-DOS 2.0引入了管道的概念，所以需要有临时文件夹来存放前一个程序的输出，作为第二个程序的输入。MS-DOS的作者选择了使用`TEMP`来作为临时文件夹的环境变量。当然了，MS-DOS怎么做并不影响别的程序员的行为，还是有人使用`TMP`。所以很多程序员就想在找临时文件夹时同时看这两个环境变量，比如`DISKCOPY`和`EDIT`就会先看`TEMP`，再看`TMP`。

Windows做了差不多的事情，只不过Windows的`GetTempFileName`函数的作者决定先看`TMP`，再看`TEMP`！！！所以现在Windows上的程序员如果使用`GetTempFileName`函数的话，我们找到的的是`TMP`这个环境变量。
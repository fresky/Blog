title: 如何创建一个有System用户权限的命令行
date: 2015-09-24 10:26:49
categories:
tags: Tool
description:
---
电脑上有个文件夹删不掉，右键属性，查看安全页面，显示用户组属于SYSTEM，就算是Administrator权限也不能删除，很是郁闷。

找到了一篇文章[Running CMD.EXE as Local System](http://blogs.msdn.com/b/adioltean/archive/2004/11/27/271063.aspx)，可以打开一个SYSTEM用户权限的命令行，那就可以用`rmdir`命令删除了。方法如下：

1.首先创建一个服务，这个服务运行的程序就是命令行`cmd`。  
```
sc create testsvc binpath= "cmd /K start" type= own type= interact
```

2.然后运行这个服务。

```
sc start testsvc
```

会报一个如下的错误，但是命令行窗口还是可以打开。

```
[SC] StartService FAILED 1053:

The service did not respond to the start or control request in a timely fashion. 
```

3.在这个新打开的命令行窗口里，就拥有了SYSTEM用户的权限。如果用完想删除这个服务，可以运行如下命令：

```
sc delete testsvc
```
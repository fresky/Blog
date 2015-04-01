---
layout: post
title: "如何定制Windows系统右键菜单"
date: 2013-06-28
comments: true
tags: Other
---
<p>今天心血来潮把几个自己常用的工具定制到了系统的右键菜单。包括notepad++，7zip，还有复制文件全路径和文件夹路径。下面简单介绍一下步骤。</p>  

#1. Windows系统右键菜单对应的注册表位置

Windows系统的右键菜单对应着如下的注册表位置。
##1）所有文件的右键菜单：

<p><a href="http://images.cnitblog.com/blog/163228/201306/28180933-152759b897fe4977b209b13f33bb1069.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://images.cnitblog.com/blog/163228/201306/28180934-a0f7aa9dd8db455589982b7c9b288f30.png" width="244" height="212" /></a></p> 

##2）所有目录的右键菜单：

<p><a href="http://images.cnitblog.com/blog/163228/201306/28180935-09f19fa288304de48ca38d8ed3d28959.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://images.cnitblog.com/blog/163228/201306/28180935-c0df8dc65c8d403f9f42e82fbed45099.png" width="308" height="116" /></a></p>  

#2.添加自己定制的右键菜单
##1）如下的代码存为.reg文件，双击就能把注册表键值导入到注册表中。


```
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\*\shell\Notepad++]
@="Notepad++"
"Icon"="...\\notepad++ico.ico"
"Position"="Middle"

[HKEY_CLASSES_ROOT\*\shell\Notepad++\command]
@="\"...\\notepad++.exe\" \"%1\""


[HKEY_CLASSES_ROOT\*\shell\7zip]
@="7zip"
"Icon"="...\\7zipico.ico"
"Position"="Middle"

[HKEY_CLASSES_ROOT\*\shell\7zip\Command]
@="\"...\\7z.exe\" a \"%1\".zip \"%1\""

[HKEY_CLASSES_ROOT\Directory\shell\7zip]
@="7zip"
"Icon"="...\\7zipico.ico"
"Position"="Middle"

[HKEY_CLASSES_ROOT\Directory\shell\7zip\command]
@="\"...\\7z.exe\" a \"%1\".zip \"%1\""


[HKEY_CLASSES_ROOT\*\shell\CopyFileFullName]
@="Copy File FullName"
"Icon"="...\\fullpath.ico"
"Position"="Middle"

[HKEY_CLASSES_ROOT\*\shell\CopyFileFullName\Command]
@="...\\copyfullname.bat \"%1\""


[HKEY_CLASSES_ROOT\*\shell\CopyFileName]
@="Copy File Name"
"Icon"="...\\name.ico"
"Position"="Middle"

[HKEY_CLASSES_ROOT\*\shell\CopyFileName\Command]
@="...\\copyname.bat \"%1\""


[HKEY_CLASSES_ROOT\*\shell\CopyFolderName]
@="Copy Folder Name"
"Icon"="...\\folder.ico"
"Position"="Middle"

[HKEY_CLASSES_ROOT\*\shell\CopyFolderName\Command]
@="...\\copyfolder.bat \"%1\""


[HKEY_CLASSES_ROOT\Directory\shell\CopyFolderName]
@="Copy Folder Name"
"Icon"="...\\folder.ico"
"Position"="Middle"

[HKEY_CLASSES_ROOT\Directory\shell\CopyFolderName\command]
@="...\\copyname.bat \"%1\""
```





##2）如下的代码是删除上面添加这些注册表键值。</p>

```
Windows Registry Editor Version 5.00

[-HKEY_CLASSES_ROOT\*\shell\Notepad++]

[-HKEY_CLASSES_ROOT\*\shell\7zip]

[-HKEY_CLASSES_ROOT\Directory\shell\7zip]

[-HKEY_CLASSES_ROOT\*\shell\CopyFileFullName]

[-HKEY_CLASSES_ROOT\*\shell\CopyFileName]

[-HKEY_CLASSES_ROOT\*\shell\CopyFolderName]

[-HKEY_CLASSES_ROOT\Directory\shell\CopyFolderName]
```



##3）简单说明

<p>其实就是把当前的文件或者文件夹作为参数（%1）传给你需要的应用。各个应用的命令行参数可以自己去查帮助。比如我的7zip使用的是压缩命令，命令行参数就是</p>

```bat
7z.exe a “%1".zip "%1"
```


<p>把当前文件或者文件夹放入名为文件（夹）名加上.zip的压缩包中。</p>

<p>&#160;</p>

<p>关于复制文件名和文件夹名的命令，我是用了如下的bat文件，分别存在了copyfullname.bat</p>

```bat
@echo off
echo %~1 | clip
```

<p>copyname.bat</p>

```bat
@echo off
echo %~nx1 | clip
```

<p>copyfolder.bat三个bat中。</p>

```bat
@echo off
echo %~dp1 | clip
```

<p>其实就是把当前参数放进了剪切板里。</p>

<p>下面列举了关于%1的一些常见用法。</p>

| 参数 | 解释 |
|--------|--------|
|%~1|删除引号(")，扩充 %1|
|%~1|删除引号(")，扩充 %1|
|%~f1|将 %1 扩充到一个完全合格的路径名|
|%~d1|仅将 %1 扩充到一个驱动器号|
|%~p1|仅将 %1 扩充到一个路径|
|%~n1|仅将 %1 扩充到一个文件名|
|%~x1|仅将 %1 扩充到一个文件扩展名|
|%~s1|扩充的路径指含有短名|
|%~a1|将 %1 扩充到文件属性|
|%~t1|将 %1 扩充到文件的日期/时间|
|%~z1|将 %1 扩充到文件的大小|


# 3. 效果



<p><a href="http://images.cnitblog.com/blog/163228/201306/28180935-73b2aea7a1ba46f6abbdef4606806ff9.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://images.cnitblog.com/blog/163228/201306/28180936-fb87d3a0b6c1421e83d1daa4b8734eb7.png" width="523" height="201" /></a></p>
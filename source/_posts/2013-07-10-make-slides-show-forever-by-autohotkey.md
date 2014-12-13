---
layout: post
title: "用Autohotkey让powerpoint幻灯片一直播放"
date: 2013-07-10
comments: true
tags: Tool
---
有台电脑专门接了个大电视循环播放一个幻灯片，但是有时候会弹出一些对话框，比如windows要更新之类的，这样的话powerpoint就不是active的进城了，这样幻灯片就会停下来，还需要人去手动点一下，非常麻烦。

可以用[autohotkey](http://www.autohotkey.com/)的脚本来解决这个问题，安装完autohotkey之后，写一个新的脚本，内容如下：

```
SetTitleMatchMode 1
IfWinExist PowerPoint Slide Show
	WinActivate
;else
	;Run "powerpointpath/POWERPNT.EXE" /s "slidespath"
return
```

这个脚本用到了autohotkey的三个命令：

[SetTitleMatchMode](http://www.autohotkey.com/docs/commands/SetTitleMatchMode.htm)，设置窗口标题的匹配模式，这里设为1意思是窗口的标题以给定字符串开始。

[IfWinExist / IfWinNotExist](http://www.autohotkey.com/docs/commands/IfWinExist.htm)，判断正在播放幻灯片的窗口是否存在。

[WinActivate](http://www.autohotkey.com/docs/commands/WinActivate.htm)，如果存在，就激活这个窗口，这样幻灯片就会继续播放了。

当然，也可以在上面的那个else里面写上如果这个窗口不存在就运行powerpoint，并且播放幻灯片。从[Command-line switches for PowerPoint](http://office.microsoft.com/en-us/powerpoint-help/command-line-switches-for-powerpoint-2007-and-the-powerpoint-viewer-2007-HA010153889.aspx)可以找到powerpnt.exe的命令行参数，给个/s表明要播放幻灯片。
---
layout: post
title: "怎么让Visual Studiot在遇到一个project编译错误时停止编译其它project"
date: 2013-01-22
comments: true
tags: Tool
---
Visual studio一个solution中含有多个project时，当编译整个solution，如果一个project编译出错，visual studio会继续编译其它的project，这在大多数情况下都是不需要的，只是浪费了时间。<br /><a href="http://visualstudiogallery.msdn.microsoft.com/91aaa139-5d3c-43a7-b39f-369196a84fa5">StopOnFirstBuildError extension</a>是一个Visual Studio2012 和2010的插件，可以让Visual Studiot在遇到一个project编译错误时停止编译其它project。对于2008，可以用<a href="http://www.csharpfritz.com/blog/stop-that-build-dont-waste-time-building-projects-in">Stop that Build! Don't waste time building projects in Visual Studio</a>的方法。<br />打开菜单：Tools-&gt;Macros-&gt;Macros IDE，双击"MyMacros",在EnviromentEvents中加入如下代码：

```vbnet
    Private Sub BuildEvents_OnBuildProjConfigDone(ByVal Project As String, ByVal ProjectConfig As String, ByVal Platform As String, ByVal SolutionConfig As String, ByVal Success As Boolean) Handles BuildEvents.OnBuildProjConfigDone

        '-- When the build of a project fails
        If (Not Success) Then

            '-- Cancel the remaining builds in the Solution
            DTE.ExecuteCommand("Build.Cancel")

            '-- Write a polite message to the OutputWindow that's logging all information about this build
            DTE.ToolWindows.OutputWindow.ActivePane.OutputString(vbCrLf & vbCrLf & "Build failed while compiling " & Project)


        End If

    End Sub
```
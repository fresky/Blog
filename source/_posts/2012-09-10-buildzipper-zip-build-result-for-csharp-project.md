---
layout: post
title: "BuildZipper——给c#项目添加postbuildevent，每次build完把build结果做个zip放在solution目录。"
date: 2012-09-10
comments: true
tags: [CSharp, Tool]
---
最近几次发布项目到github上，都手动把最后build的结果做个zip，传上去，供直接下载binary用。觉得很费事，就写了个小程序，BuildZipper。<br />源代码在<a href="https://github.com/fresky/BuildZipper">github</a>上，运行文件<a href="https://github.com/fresky/BuildZipper/blob/master/BuildZipper.zip">这里</a>下载。<br />用了<a href="http://dotnetzip.codeplex.com/">DotNetZip Library</a>来做zip，用了C#的<a href="http://msdn.microsoft.com/en-us/library/microsoft.build.evaluation.project.aspx">Project</a>来操作project文件。<br /><blockquote></blockquote>
---
layout: post
title: "Windows Live Writer的Markdown插件"
date: 2013-07-16
comments: true
tags: CSharp
---
<p>我新写了一个Windows Live Writer的Markdown插件，代码放在了<a href="https://github.com/fresky/MarkdownInLiveWriter">github</a>上。</p>
<h3>介绍</h3>
<p>这个项目是一个<a href="http://windows.microsoft.com/en-us/windows-live/essentials-other#essentials=overviewother">Windows Live Writer</a>的Markdown插件。有了这个插件，你可以用Markdown来写你的blog，同时可以看到实时的预览。</p>
<p>这个插件的Markdown处理器使用了<a href="http://code.google.com/p/markdownsharp/">markdownsharp</a>，它是一个开源的Markdown处理器的C#实现。</p>
<h3>安装</h3>
<p>一下两种方法都行。</p>
<p>1. 下载 <a href="https://github.com/fresky/MarkdownInLiveWriter/blob/master/MarkdownInLiveWriter.dll">MarkdownInLiveWriter.dll</a>，然后放到 [WindowsLiveWriterPath]\Plugins\目录下。</p>
<p>2. 下载 <a href="https://github.com/fresky/MarkdownInLiveWriter/blob/master/MarkdownInLiveWriter.msi">MarkdownInLiveWriter.msi</a>，然后运行安装文件。</p>
<h3>使用</h3>
<ol>
<li>在Windows Live Writer中点击<strong>插入</strong>菜单，然后点击<strong>Insert Markdown</strong>。</li>
<li>在弹出来的控件左边写Markdown，然后右边就会实时的预览。</li>
<li>也可以点击右下角的<strong>Source</strong>，来看生成的html代码。</li>
<li>点击<strong>Insert</strong>来插入到Windows Live Writer中。</li>
</ol>
<p>参见下图：</p>
<p><a href="http://images.cnitblog.com/blog/163228/201307/16195559-3bb304db41184f86a4d3480955a190e1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://images.cnitblog.com/blog/163228/201307/16195601-f2374f822814416fbd5698a31401e702.png" alt="image" width="733" height="514" border="0" /></a></p>
<h3>下一步工作</h3>
<ol>
<li>更好的支持代码高亮，把我的另一个Windows Live Writer的代码语法高亮的插件[CodeInLiveWriter](/2013/06/09/codeinlivewriter-code-syntax-highlight-plugin-for-windows-live-writer/)集成进来。</li>
</ol>
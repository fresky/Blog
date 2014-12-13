---
layout: post
title: "使用Ready2Search来定制Firefox和Chrome的搜索框"
date: 2014-04-30 13:07
comments: true
tags: Tool
keywords: Ready2Search, firefox, chrome, search plugin
description: 使用Ready2Search来定制Firefox和Chrome的搜索框
---
我一直是用Firefox作为我的主浏览器，可以在Firefox中定制搜索插件，这样就能在地址栏很方便敲搜索关键词调用相应的搜索插件。以前都是手动的写`xml`文件来加搜索插件，今天发现了一个网站[Ready2Search](http://ready.to/search/en/)可以很方便的自动定制Firefox或者Chrome的搜索框。

使用很简单，打开网站[Ready2Search](http://ready.to/search/en/)，输入名字和搜索链接，以豆瓣搜索为例，链接就是`http://www.douban.com/search?q=`。在网站右边还能添加图标，我就是直接下载豆瓣的[Favicon](http://img3.douban.com/favicon.ico)。然后点击`Make Search plug-in`按钮就行了。

原理很简单，就是在Firefox的`profile\searchplugins\`目录下增加一个搜索的`douban.xml`。

生成的豆瓣对应的文件内容如下：
```xml
<SearchPlugin xmlns="http://www.mozilla.org/2006/browser/search/" xmlns:os="http://a9.com/-/spec/opensearch/1.1/">
<os:ShortName>Douban</os:ShortName>
<os:Description>Douban search</os:Description>
<os:InputEncoding>UTF-8</os:InputEncoding>
<os:Image width="16" height="16">data:image/x-icon;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAIAAACQkWg2AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAADlSURBVDhPY7CY6MpQLkgkAilGEyKMgNhjVvB/IgBQGVSDUot+3bY2ggioDKoBiATKpIhBCA3WE9yBlv779w9EoiIIACqAqETRcOfNPb95IZ7TPT1nB3jOCvCc6Q1kAwXhGkCWIGt4/Ok5f4kUGrr44jpODUDD9Hts0BA+G/AAdA1AlvtMUGz8+fdXr8PKrNcZiIAMoMjXX9+BUiClaBqA6N7bB3/+/kH2AFADUBCuAKEBgrr3TwaqyFhZBFENZAC5QEGILELD5MOzgBIEAVAZSCcQhy9OWnZmDUEEVAbSQFrynugKAKSaaH1V3D9GAAAAAElFTkSuQmCC</os:Image>
<os:Url type="text/html" method="GET" template="http://www.douban.com/search?q={searchTerms}">
</os:Url>
</SearchPlugin>
```
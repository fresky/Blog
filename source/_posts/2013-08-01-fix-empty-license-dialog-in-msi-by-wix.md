---
layout: post
title: "解决wix生成的msi的license对话框空白的问题"
date: 2013-08-01
comments: true
tags: Installer
---
今天用Wix做之前写的那个[Windows Live Writer的Markdown插件](http://fresky.github.io/blog/2013/07/16/windows-live-writer-plugin-markdowninlivewriter/)的msi安装包，在wxs文件中用如下的代码添加license文件，结果发现生成msi后license文件框一直是空白。

```
<WixVariable Id="WixUILicenseRtf" Value="path\License.rtf" />
```

google后才发现这是Wix的一个[bug](http://scribefire-next/wix.sourceforge.net/manual-wix3/WixUI_customizations.htm)，必须用wordpad来存 rtf文件，不能用word。参见stackoverflow上的这个[回答](http://stackoverflow.com/questions/6380724/wix-specify-licence-shows-nothing)
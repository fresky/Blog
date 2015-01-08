title: Firefox 34的"Manage Search Engine"去哪了？
date: 2015-01-08 18:22:38
categories:
tags: Other
description: 本文介绍了如何在Firefox的最新版本34上打开”Manage Search Engine“的按钮，这样在新加搜索引擎后就可以继续设置关键字了。
---
我在之前的博客[使用Ready2Search来定制Firefox和Chrome的搜索框](/2014/04/30/use-ready2search-to-customize-your-search-plugin-in-firefox-and-chrome/)介绍过如何定制Firefox的搜索框，在设置完关键字之后就能直接在地址栏选择搜索引擎，非常方便。但是我更新了Firefox的最新版本34之后，发现搜索框被“加强”了。没有那个”Manage Search Engine“的按钮，这样在新加搜索引擎后就不能设置关键字了。

解决办法是在firefox中打开这个地址[chrome://browser/content/search/engineManager.xul](chrome://browser/content/search/engineManager.xul)，这样就可以看到那个熟悉的界面了。
---
layout: post
title: ".gitignore模板"
date: 2013-06-27
comments: true
tags: Tool
---
<p><a href="https://github.com/github/gitignore">github/gitignore &middot; GitHub</a>列举了一些有用的.gitignore的模板。比如<a href="https://github.com/github/gitignore/blob/master/VisualStudio.gitignore">这个</a>是visual studio的。</p><p>另外说一个题外话，如果不想看见solution目录的那个sdf，（visual studio生成的intelligence数据库.sdf，取代了ncb），可以通过Tools -&gt; Options -&gt; Text Editor -&gt; C/C++ -&gt; Advanced，然后在 "Fallback Location"中设置 "Always Use Fallback Location" 和"Do Not Warn If Fallback Location Used" <code>为True。在</code> "Fallback Location" 留着空白就行，这样VS会把sdf生成到临时文件夹中。</p>
---
layout: post
title: "OSQA不能搜索question的解决办法。"
date: 2012-10-26
comments: true
tags: Tool
---
<p>昨天搭建好的OSQA，发现不能搜索question，只能搜索tag和user，搜索后发现在settings_local.py中的disabled modules里面加上 mysqlfulltext 就好了。但是这样就不能全文搜索了，只能支持问题的搜索。</p>
<p><br />还得在研究研究。</p>
<div><embed id="lingoes_plugin_object" width="0" height="0" type="application/lingoes-npruntime-capture-word-plugin" hidden="true" /></div>
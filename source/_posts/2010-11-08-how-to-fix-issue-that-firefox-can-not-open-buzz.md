---
layout: post
title: "firefox 3.6.12 不能打开gmail的buzz，setting和chat 解决办法"
date: 2010-11-08
comments: true
tags: Other
---
前段时间升级firefox到3.6.12后发现不能打开gmail里面的buzz，setting和chat了，今天网上搜了搜，找到了解决办法。在地址栏输入about:config进入firefox设置，找到Dom.storage.enabled，设置成true就行了。<br />详细请看<a href="https://support.mozilla.com/en-US/questions/763190#answer-115496">Some parts of GMAIL no longer work after a recent update</a> 和<a href="http://kb.mozillazine.org/Dom.storage.enabled"> Dom.storage.enabled - MozillaZine Knowledge Base</a> 。<br /><br />
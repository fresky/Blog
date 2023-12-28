---
layout: post
title: "http的url长度限制"
date: 2011-01-11
comments: true
tags: Programming
---
http的url长度是有限制的，<a href="http://www.boutell.com/newfaq/misc/urllength.html">WWW FAQs: What is the maximum length of a URL?</a><br />所以不能用get的方式时在url里面放太多的东西，举个简单的例子。<br />如果一个asp.net的页面中使用了treeview，并且treeview的enableviewstate是true（默认情况）。但是如果你不小心在form中设置成了get时，你会发现只添加6、7个节点就会出错（IE，不同浏览器的长度限制不一样），页面不能访问，因为如果用get，viewstate会放到http的url中，会超出url的最长限制。
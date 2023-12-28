---
layout: post
title: "VS2012的自动生成测试的插件 Unit Test Generator"
date: 2013-08-19
comments: true
tags: [Programming, Tool, Testing]
---
<p><a href="http://visualstudiogallery.msdn.microsoft.com/45208924-e7b0-45df-8cff-165b505a38d7">Unit Test Generator extension</a>是一个VS2012的插件，可以为C#的public方法很方便的自动生成unit test。安装这个插件后点击TEST菜单可以配置，如下所示：</p>  <p><a href="http://images.cnitblog.com/blog/163228/201308/19140430-538d7d57193049bc8bf91a7095c1b569.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://images.cnitblog.com/blog/163228/201308/19140431-8d614d00b6534ee7904949c370eb2451.png" width="244" height="188" /></a></p>  <p>在配置里可以设置测试框架和自动生成的项目，类，方法名字以及默认的测试函数实现。</p>  <p><a href="http://images.cnitblog.com/blog/163228/201308/19140431-0f7b899f39a04560a25abda6d81a52f8.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://images.cnitblog.com/blog/163228/201308/19140432-d17999dc54f8486cabf242589696c9c3.png" width="446" height="207" /></a></p>      <p>然后在C#的public的方法上点右键，就能自动生成测试了，会帮你自动创建一个test项目，非常方便。</p>
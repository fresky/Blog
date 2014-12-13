---
layout: post
title: "根据代码的现状调整估计"
date: 2012-10-11
comments: true
tags: Development
---
<a href="http://www.mlibby.com/grading-code/">Grading Code</a>这篇文章说在legacy code（遗留代码）中做开发时估计要根据现有代码的状况来做个调整了，给了如下11个评估标准：<br /><ol><li>代码有单元测试，并且测试可以跑过。</li><li>单元测试覆盖了主要的应用逻辑。</li><li>业务逻辑和数据存储逻辑分开。</li><li>业务逻辑和显示逻辑分开。</li><li>大系统中代码被合适的分到了不同的项目中。</li><li>代码编译没有warning和error。</li><li>代码符合某种编码规范。</li><li>没有长方法。（作者认为应该小于32行，如果大于64行就是长方法，如果在32-64行之间，不能有超过2层if/for等。</li><li>封装。在OO中行为属于自己的类。</li><li>没有滥用pattern。比如只有一个strategy时用strategy pattern。</li><li>没有在该用pattern的时候没有用pattern。<br /></li></ol>
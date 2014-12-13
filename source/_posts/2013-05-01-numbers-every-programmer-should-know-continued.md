---
layout: post
title: "每个程序员都需要知道的数字（续）"
date: 2013-05-01
comments: true
tags: Algorithm
---
<p><a href="http://highscalability.com/numbers-everyone-should-know">High Scalability - High Scalability - Numbers Everyone Should Know</a>列举了另外一些数字：</p>  <h3>写很昂贵</h3>  <ul>   <li>写是transactional的</li>    <li>disk access意味着disk seek</li>    <li>经验法则：disk seek需要10毫秒</li>    <li>算一算：1s/10ms = 100 seeks/s，每秒最多100次disk seek</li>    <li>和数据的大小的形状有关系，批量处理。</li> </ul>  <h3>读很便宜</h3>  <ul>   <li>都不需要transaction</li>    <li>数据从磁盘中读取一次，很容易就被缓存</li>    <li>所有后续的读取都是从内存来的</li>    <li>经验法则：内存中读取1m的数据需要250微秒</li>    <li>算一算：1s/250usec=4g/sec，美妙最多读4g的数据。对于1m的数据，每秒可以取4000次</li> </ul>  <h3>一些数字</h3>  <ul>   <li>L1缓存寻址需要0.5纳秒</li>    <li>branch mipredict需要5纳秒</li>    <li>L2缓存需要7纳秒</li>    <li>mutex lock/unlock需要100纳秒</li>    <li>内存寻址需要100纳秒</li>    <li>压缩1k的数据需要10000纳秒</li>    <li>通过1gbps的网络发送2k的数据需要20000纳秒</li>    <li>内存中读取连续的1m数据需要250000纳秒</li>    <li>round trip within same datacenter需要500000纳秒</li>    <li>硬盘须知需要10000000纳秒</li>    <li>从网络上读1m的连续数据需要10000000纳秒</li>    <li>从磁盘上都1m的连续数据需要30000000纳秒</li> </ul>  <p>&#160;</p>  <h3>小结</h3>  <ul>   <li>写比读昂贵40倍</li>    <li>全局共享的数据很昂贵。这是分布式系统的瓶颈。</li>    <li>系统设计时就要考虑写操作的可扩展性。</li>    <li>优化以减少写竞争。</li>    <li>让写操作尽可能地并行。</li> </ul>
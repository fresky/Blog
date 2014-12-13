---
layout: post
title: "数据库事务的ACID和BASE"
date: 2013-07-25
comments: true
tags: Database
---
<p><a href="http://www.johndcook.com/blog/2009/07/06/brewer-cap-theorem-base/">ACID versus BASE for database transactions</a>解释了ACID和BASE的区别。如下：</p>  <p>ACID: （关系数据库）</p>  <ul>   <li><strong>A</strong>tomic: 原子性，一个事务要么全部成功，要么全部回滚。 </li>    <li><strong>C</strong>onsistent: 一致性，完成一个事务后数据库不能处在一个不一致的状态。</li>    <li><strong>I</strong>solated: 隔离性，事物之间不能互相影响。</li>    <li><strong>D</strong>urable: 持久性，完成的事务对数据库所作的更改便持久地保存在数据库之中。</li> </ul>  <p>以为CAP不能兼得（参见<a href="http://www.julianbrowne.com/article/viewer/brewers-cap-theorem">Brewer's CAP Theorem</a>），</p>  <ul>   <li><strong>C</strong>onsistency：一致性。</li>    <li><strong>A</strong>vailability：可用性。</li>    <li><strong>P</strong>artition tolerance：分区容忍性。</li> </ul>  <p>所以NoSql的采用BASE事务。</p>  <p>BASE: </p>  <ul>   <li><strong>B</strong>asic <strong>A</strong>vailability：基本可用。</li>    <li><strong>S</strong>oft-state ：柔性事务。</li>    <li><strong>E</strong>ventual consistency：最终一致性。</li> </ul>
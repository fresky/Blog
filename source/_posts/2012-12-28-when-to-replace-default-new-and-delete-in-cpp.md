---
layout: post
title: "什么情况下要替换C++自带的new和delete"
date: 2012-12-28
comments: true
tags: Programming
---
<a href="http://stackoverflow.com/questions/7149461/why-would-one-replace-default-new-and-delete-operators">c++ - Why would one replace default new and delete operators? - Stack Overflow</a><br /><br /><ol><li>用来检测用户错误，比如（1）new的时候可以记录所有new出来的地址，然后用户忘记delete时帮用户delete（2）new出地址时前后放一些记号，防止overrun和underrun</li><li>用来提高效率</li><li>用来收集统计数据，比如（1）地址分布，生存期分布，分配顺序，内存使用在时间上的变化（2）统计一个类生成了多少个对象，或者限制</li><li>补偿内存对齐</li><li>把相关的对象地址放在一起</li><li>实现非常规的行为，比如delete后设为0</li></ol><blockquote></blockquote>
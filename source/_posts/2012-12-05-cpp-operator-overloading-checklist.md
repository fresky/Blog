---
layout: post
title: "C++中运算符重载需要注意什么？"
date: 2012-12-05
comments: true
tags: CPP
---
<p><a href="http://stackoverflow.com/questions/4421706/operator-overloading">c++ - Operator overloading - Stack Overflow</a>这篇FAQ讲的很清楚，把要点整理如下：</p>  <h2>C++中operator重载的基本语意：</h2>  <ol>   <li>只能重载用户定义类型的operator。</li>    <li>以下operator不能重载：.,::,sizeof,?:</li>    <li>其余的都能重载，分别是：</li>    <ol>     <li>算数运算符：二元：+ - * / %&#160; += -= *= /= %=，一元：+ - ++ --</li>      <li>位运算符：二元 &amp; | ^ &lt;&lt; &gt;&gt; and &amp;= |= ^= &lt;&lt;= &gt;&gt;= ;一元 ~</li>      <li>布尔运算符：二元：== != &lt; &gt; &lt;= &gt;= || &amp;&amp;， 一元！</li>      <li>地址管理：new new[] delete delete[]</li>      <li>显示转换运算符</li>      <li>其它：二元：= [] -&gt;，一元：* &amp;，函数调用： ()</li>   </ol> </ol>  <h2>三个基本原则：</h2>  <ol>   <li>如果operator的含义模糊不清，就不要重载，用一个函数名清楚的函数替代</li>    <li>永远坚持operator众所周知的语意</li>    <li>永远提供相关的运算符重载，比如重载了+，就要重载+=</li> </ol>  <h2>运算符实现成员还是非成员</h2>  <ol>   <li>赋值运算符=，数组下标运算符[]，成员访问运算符-&gt;和函数调用运算符()必须是成员内的。</li>    <li>如果需要修改左操作数，通常实现在非成员，比如&lt;&lt;和&gt;&gt;。</li>    <li>对于别的，遵守下面的规则：</li>    <ol>     <li>如果是一元运算符，成员</li>      <li>如果二元运算符，左右操作数对等，非成员</li>      <li>如果二元运算符，左右操作数不对等，成员</li>   </ol> </ol>
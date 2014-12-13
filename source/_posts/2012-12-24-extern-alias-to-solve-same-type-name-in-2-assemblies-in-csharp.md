---
layout: post
title: "C#用extern alias解决两个assembly中相同的类型全名"
date: 2012-12-24
comments: true
tags: CSharp
---
<p>如果你使用到的第三方库中有2个assembly中出现了完全一样的类型，C#中可以用<a href="http://http://msdn.microsoft.com/en-us/library/ms173212(v=vs.110).aspx">extern alias</a>来解决。</p>  <p>比如grid.dll和grid20.dll中都有一个类 Grid.SmallGrid，在我们的代码中必须通过命令行编译程序，</p>    

```
csc /r:GridV1=grid.dll /r:GridV2=grid20.dll mygrid.cs
```


<p>在mygrid.cs中就能用extern alias了。</p>


```c#
extern alias GridV1;
extern alias GridV2;
GridV1::Grid.SmallGrid …
GridV2::Grid.SmallGrid …
```
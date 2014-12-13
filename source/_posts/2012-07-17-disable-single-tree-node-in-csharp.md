---
layout: post
title: "禁用C#的树的单个节点"
date: 2012-07-17
comments: true
tags: CSharp
---
C#的tree 控件有Enabled属性，可以用来确定整棵树能不能选。但是如果想只禁用掉某个节点，treenode上就没有Enabled这个属性了。可以用下面的方法来实现。

```c#
private void Tree_BeforeSelect(object sender, TreeViewCancelEventArgs e)
{
    e.Cancel = !IsTreeNodeSelectable(e.Node);
}
```
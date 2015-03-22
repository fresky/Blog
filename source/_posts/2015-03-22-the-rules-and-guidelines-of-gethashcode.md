title: 实现GetHashCode时要遵循的规则
date: 2015-03-22 23:45:17
categories:
tags: CSharp
description: 本文介绍了Eric Lippert给出的实现GetHashCode时需要遵循的规则。
---

Eric Lippert的[Guidelines and rules for GetHashCode](http://blogs.msdn.com/b/ericlippert/archive/2011/02/28/guidelines-and-rules-for-gethashcode.aspx)介绍了实现GetHashCode时需要遵循的规则和指导原则，摘要如下：

规则1：相等的对象必须要有一样的哈希值。

规则2：GetHashCode返回的整数在这个对象存储在一个依赖哈希值的数据结构中不能变化。

规则3：GetHashCode的使用者不能假定哈希值不会随着时间变化，也不能假定跨AppDomain之后哈希值还保持不变。

规则4：GetHashCode永远不能抛异常，而且必须返回。

指导原则1：GetHashCode的实现必须非常快。

指导原则2：哈希值的分布必须是随机的。

安全因素1：如果攻击者能够选择哈希值，那就非常危险了。

安全因素2：GetHashCode不能用来干别的事情，它只能用来使得哈希表分布比较均匀。它不能用来1）给对象提供一个唯一的表示。2）不具备密码功能。3）不能用来做校验。
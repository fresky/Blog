title: Code Smell那么多，应该先改哪一个？
date: 2015-05-03 09:32:50
categories:
tags: Development
description: God Class和违反Interface Segregation是最严重的两个Code Smell，应该首先改善。
---
我的日常工作就是对付一个大型的遗留系统，Code Smell遍地都是，但是在有限的时间内，先修改哪些Code Semll更有意义呢？[How bad is smelly code?](http://fagblogg.mesan.no/how-bad-is-smelly-code/)是基于2009年[Simula Research Laboratory](http://www.simula.no/)的一次实验统计出来的一个关于衡量Code Smell的影响的一个报告。他们的实验方法是在历时3个月的项目中，每天访问开发者，并且在开发者的IDE中安装了插件来统计花费在修改每个文件以及搜索、浏览每个文件花费的时间，还有每个文件引入的bug。

这个统计报告的结果如下：

1. God Class非常坏！统计发现大于1844行的文件比600行以内的文件要多花费10倍时间来阅读来修改。所以我们应该把大文件分割成小文件，每个文件最好不要超过1000行。唉，想想我的代码库中动辄上万行的文件。。。  
1. Data Clumps还行，坏处不大，建议就先不管了。  
1. 要坚持接口分离（Interface Segregation）原则，建议把大的接口按照功能分到不同的小的接口。  
1. Code Smell经常组团出现，所以建议在修改一个Code Smell之后看看这个文件有没有别的Code Smell，有一个Code Smell的关系图如下。  

{% limg CodeSmellRelations.jpg %}
title: 程序员的开发效率可以衡量吗？
date: 2023-11-15 13:16:51
categories:
tags:
description:
---

前段时间随着麦肯锡的一篇文章[Yes, you can measure software developer productivity](https://www.mckinsey.com/industries/technology-media-and-telecommunications/our-insights/yes-you-can-measure-software-developer-productivity)引起了网上一堆批评，基本是说首先太难定义合理的指标来衡量，其次就算定义了指标，根据[Goodhart's law(古德哈特定律)](https://en.wikipedia.org/wiki/Goodhart%27s_law) - When a measure becomes a target, it ceases to be a good measure（当一项指标成为目标时，它就不再是一个好的指标），根据这个指标衡量也会起到不好的效果。

麦肯锡的衡量方法基本由下面组成：

* 来源于Google的[DORA (DevOps Research and Assessment)](https://dora.dev/)的四个指标：
  * Lead time - 提交改动到交付用户的时间
  * Deploy frequencey - 部署频率
  * Change fail percentage - 改动导致的失败率
  * Time to restore - 恢复用时

* 来源于Github和微软的[SPACE 模型](https://queue.acm.org/detail.cfm?id=3454124)：
  * Satisfaction and well-being
  * Performance
  * Activity
  * Communication and collaboration
  * Efficiency and flow

* 麦肯锡这次提出的Opportunity-focused Metrics：
  * Inner/outer loop time spent - 内循环和外循环的时间花费
  * Developer Velocity Index benchmark - 程序员开发速度指数基准
  * Contribution analysis - 贡献分析
  * Talent capability score - 人才能力评分

看了一堆关于程序用开发效率的文章后，觉得这篇[What makes developers productive?](https://jeremymikkola.com/posts/developer_productivity.html)总结的影响程序员开发效率的因素还挺靠谱的，把重点摘录如下。

* 知道要做的东西
* 做更少的事情。很快完成工作很好，但是完全不用做更棒。
* 反馈很快的工具
* 程序员的知识。专精很重要，不能只有广度。
* 有用的基础设施
* 低技术债
* 低失败率
* 务实的提高生产力的实践。你鼓励程序员提高效率的实践得是非常容易的，比如假设你鼓励程序员做原型来验证，就让搭建原型很容易，假设你鼓励程序员每次代码提交都很小便于代码审查，就让提交小的代码变动很容易。
* 专注。减少会议和随意打断对程序员生产力的伤害，避免让程序员脑子里同时担心很多事情。
* 做事情做完。事情做了一半的开发效率是0，并不是一半。及时止损沉没成本很重要，但是更重要的是要尽量避免半途而废。

能不能衡量开发效率放到一边，对于我们每个人来说，尽量找到方法提高自己的开发效率肯定是需要的。
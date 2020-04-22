title: 微服务带来的好处和挑战
date: 2020-04-19 15:51:07
categories:
tags: Design
description: 
---

微服务（[Microservices](https://en.wikipedia.org/wiki/Microservices)）是一个非常热门的话题，在很多场景下，采用微服务已经成了政治正确的事情。本文讨论一下微服务的好处和挑战，特别是在哪些情况下采用微服务需要慎重考虑，最后给了一些采用微服务的一些建议。

# 微服务的特点

微服务没有一个统一的定义，采用微服务的应用是由一组提供业务逻辑的服务和对这些服务的管理组成的。

通常来说微服务里的服务有如下特点：

* 服务是小的，独立的，松耦合的
* 一个小的团队可以开发和维护一个服务
* 服务可以独立的部署，一个团队可以更新已有的服务，而不需要重新编译或者部署整个应用
* 服务负责自己的数据存储
* 服务直接的交互通过定义好的API实现，内部实现细节不需要被其他服务知道
* 服务之间可以采用不同的技术栈，库或者框架

对服务的管理（通常由现成的技术和产品实现）通常包含如下部分：


* 服务编排（orchestration）。负责协调和管理内部的服务，具有如下功能：
  * 部署
  * 监控
  * 发现错误
  * 负载均衡等
* API网关。是应用对外的入口，负责把外部对应用的调用分发到合适的服务。API网关的如有下优点：
  * 外部对服务解耦，服务可以更新版本，重构不需要修改外部客户
  * 服务之间可以用其它对Web不友好的协议，比如消息队列AMQP
  * 可以做其他横切关注点(Cross cutting Concern)的功能比如认证，日志，负载均衡等

# 微服务带来的好处

从上面微服务的特点，我们可以看到微服务会带来下面的好处：

* 敏捷，更快的发布频率，更适合实现持续发布
* 更容易对服务进行扩展（可以只针对需要的服务扩展）
* 多个团队可以独立的发布服务
* 服务本身非常简单，聚焦于把一件事情做好
* 每个服务可以用最合适的工具来完成，可以混用技术栈
* 微服务架构的系统天生就是松耦合的
* 小而精的团队
* 小的代码库
* 开发团队的可扩展性，可以在大的项目中增加更多的团队，相对独立的工作
* 失败隔离
* 数据隔离


我们再回头想想我们现在大部分的项目，大多数都具有如下的情况：
* 需求变化很快，需要快速拥抱变化，快速发布产品
* 在需要的时候能够扩展（scale），最好能针对子功能扩展
* 在需要的时候希望能通过加团队的方式来推进项目进展
* 希望一个子系统的失败不会影响整个应用

对比微服务带来的好处和现在我们大部分项目的要求，感觉微服务就是为我们量身定制的。不过微服务也会带来很多挑战，下面我们看看这些挑战。

# 微服务带来的挑战

## 清晰的划分服务边界通常非常困难

需要对问题领域有清楚的认识才能正确划分服务边界，这个通常非常困难，而且经常随着开发的进展会发现需要改变服务边界。如果服务边界划分不合适就会带来技术上的复杂性，比如：
* 支持分布式事务
* 共享数据
* 服务之间交互太多等

## 微服务在设计上要考虑的问题
  
* 怎么管理横切关注点(Cross cutting Concern)
* 怎么共享数据，数据重复到什么地步
* 怎么处理共享的逻辑（作为一个独立的服务？在每个服务中重复代码？做一个共享的库？）
* 怎么管理跨服务的报表
* 是否要把UI放进服务里

## 微服务的实现和测试更复杂

* API数目成倍增加
* 引入网络延迟
* 不可靠的网络，容错处理
* 数据一致性
* 跨服务的事务处理很复杂
* 调试分布式系统很复杂
* 跨服务的重构会很困难
* 跨服务的日志会很困难
* 服务支持多个版本很难处理，特别是有状态的服务
* 很难在测试中重建和生产环境一致的环境，自动化测试带来的自信更低，更依赖基于生产环境的监控而采取合适的措施而不是发布前的测试

## 微服务的运维开销更大

* 更多的服务需要编译、测试、部署、运行，而且可能多种编程语言，运行环境
* 更多的服务需要集群来处理故障转移，负载均衡
* 需要高质量的服务监控和运维基础设施
* 对健壮而且自动化的部署要求更高

## 对团队的要求更高
* 组织结构需要转型到自治的，跨功能的团队
* 团队的技术能力，技术栈需要扩展
* 要求团队更高的DevOps水平
* 由于缺少监管更依赖于团队的自觉

# 采用微服务的一些建议

在上面列出的这些挑战中，我觉得最大的挑战是第一个**清晰的划分服务边界通常非常困难**。就像Martin Fowler说他观察到的使用微服务的模式是：
1. 几乎所有的微服务的成功故事都是始于Monolith，然后因为太大而分解成微服务  
2. 几乎所有的一开始就采用微服务的系统，都陷入了很大的麻烦

也就是说更多情况下**微服务不是天生的，是逐渐长成的**。这是因为我们通常需要时间来理解我们的问题领域才能更好的划分服务边界。如果我们采取了微服务，但是服务划分的不对，只会给我们带来更多的麻烦。所以最主要的采用微服务的建议就是：

如果我们要解决的问题领域很复杂，并且如何划分领域边界非常清楚（比如我们在重写一个巨大的遗留系统），那么我们就可以从开始就采用微服务。否则，我们就首先Monolith，但是要注意系统内的模块划分，等我们对一个模块边界清楚之后，再把这个模块分成一个微服务。

下面给出了一些问题和条件来帮助我们选择要不要采用微服务。

## 什么时候采用微服务

* 现在的Monolithic想实现子模块不同的可扩展性，灵活性或者发布频率
* 想用新的编程语言或者技术栈来重写遗留系统
* 已有应用或者模块需要被其他应用重用

## 什么时候不采用微服务

* 做一个全新的产品（领域模型不清晰，变化很大）
* 团队很小（不值得花费微服务的额外开销）

## 在权衡是否采用微服务时可以问自己的问题

* 你有Monolithic依赖吗？如果有的话能把这个依赖分解吗？不然的话可能会有性能的问题，而且领域边界更不容易确定
* 团队（应用）足够大需要分解成微服务吗？
* 领域模型足够清楚和确定，可以正确的划分领域边界吗？
* 组织结构支持微服务吗？如果还是按照前端，后端，数据库来分组的吗？
* 团队理解微服务需要的节技能吗？比如DevOps，Docker，Orchestration等等
* 需要对应用中的某一个模块做扩展吗？


## 微服务的实现中的一些危险信号
* 某一个服务开始读取其他服务的数据库
* 需要按照一定的顺序来部署或者启动服务，或者端到端的测试必须在所有的服务都部署好后才能开始

## 决定采用微服务之后的一些建议

* 除了根据业务需求，领域知识，也可以根据不同的发布周期、扩展需求、数据模式等划分服务
* 从第一天就开始自动化测试，采用统一的自动化Pipeline
* 容器化，尽量采用现成的服务管理工具，专注于业务逻辑
* 规定一些服务规范，避免太过freestyle。比如下面是一个关于微服务是每个服务一个代码库还是共用一个代码库的比较。


|| Monorepo|Multiple repos|
|---|---|---|
|优点|代码分享<br/>容易标准化代码和共享工具<br/>容易重构代码<br/>可发现性高|清晰的代码所有权<br/> 更少的合并冲突<br/>帮助强制实现微服务之间的解耦|
|缺点|修改共享代码会影响多个微服务<br/>更大可能出现合并冲突<br/>工具必须能够支持大代码<br/>访问控制<br/>部署流程复杂|更难共享代码<br/>更难强制代码规范<br/>依赖管理<br/>可发现性低<br/>缺少共享的基础设施|


# 参考阅读

本文的很多观点来自下面这些文章：  

* 微服务的一些权衡
 
[Should I use microservices?](https://www.oreilly.com/content/should-i-use-microservices/)  
[Microservice Premium](https://martinfowler.com/bliki/MicroservicePremium.html)  
[The Death of Microservice Madness in 2018](https://dwmkerr.com/the-death-of-microservice-madness-in-2018/)  
[Microservices without fundamentals](https://yermakov.net/microservices-without-fundamentals/)  
[Microservices — Good or Bad?](https://r2m.se/microservices-good-or-bad/)  
[Microservice Prerequisites](https://martinfowler.com/bliki/MicroservicePrerequisites.html)  
[When Not To Use Microservices](https://www.feval.ca/posts/microservices/)  
[STOP!! You don’t need Microservices.](https://medium.com/swlh/stop-you-dont-need-microservices-dc732d70b3e0)  
[About When Not to Do Microservices](https://blog.christianposta.com/microservices/when-not-to-do-microservices/)  
[When To Use – and Not To Use – Microservices](https://containerjournal.com/topics/container-ecosystems/when-to-use-and-not-to-use-microservices/)   
[Monolith vs Microservices](https://programmerfriend.com/monolith-vs-microservices/)  
[Microservices - Not a free lunch!](http://highscalability.com/blog/2014/4/8/microservices-not-a-free-lunch.html)  

* 实现微服务的一些建议

[Monolith First](https://martinfowler.com/bliki/MonolithFirst.html)  
[Building microservices on Azure](https://docs.microsoft.com/en-us/azure/architecture/microservices/)  
[CI/CD for microservices architectures](https://docs.microsoft.com/en-us/azure/architecture/microservices/ci-cd)  
[Clean Micro-service Architecture](https://blog.cleancoder.com/uncle-bob/2014/10/01/CleanMicroserviceArchitecture.html)  
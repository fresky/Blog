title: 常见的代码分支模型和比较
date: 2020-03-10 13:54:45
categories:
tags: Development
description: 这篇文章介绍一下常用的代码分支模型，包括Git Flow，GitHub Flow和Trunk Based Devlopment。
---

怎么做代码分支管理是每一个项目需要面临的问题，我认为代码分支管理是由产品发布模型决定的，在满足产品发布模型的同时要尽量减少开发、测试和部署的工作量。这篇文章简单介绍一下常见的几种代码分支模型，希望能给大家怎么选择适合自己的代码分支模型带来一些帮助。

# Git Flow

**Git Flow**是在2010年的这篇文章[A successful Git branching model - GitFlow](https://nvie.com/posts/a-successful-git-branching-model/)中提出来了，之后被广泛采用，它的模型如下所示。

{% limg gitflow.png %}

**Git Flow**有如下要点：
1. 主分支有两个：**master**是部署分支，所有已经发布的版本都在这里。**develop**是开发分支。
1. 有三种支持分支：**feature**用来做功能开发，**release**用来准备下一个要发布的版本，**hotfix**用来在已经发布的版本上打补丁。
1. 对于测试来说，必须在**release**分支和**hotfix**分支做严格的发布前的测试。
1. 在**master**分支上几乎不会有冲突。

Git Flow要求所有支持分支（**feature**，**release**和**hotfix**）都需要把代码merge回**master**。这也是这个分支主要要避免的风险。

Git Flow的分支模型非常清晰，可以很容易的支持各种产品发布策略，但是并不是说所有的项目都应该直接照搬这个模型，因为这么多分支本身就会带来管理上的复杂性。对我来说，这个模型更像是一个基础的代码分支模型，它帮助我们思考每一种分支存在的意义，从而帮助我们找到适用于我们自己的产品发布策略的分支模型。下面我们分别来看看这些分支存在的意义。

## Hotfix分支

在讨论**hotfix**分支之前，我想先讨论几个概念。第一个是**产品支持阶段**，我认为**产品支持阶段**=**产品退休日期**-**产品发布日期**。第二个概念是**可能的hotfix阶段**，我认为**可能的hotfix阶段**=**产品支持阶段**-**hotfix开发测试时间**。只有在**可能的hotfix阶段**大于0的时候，才需要存在**hotfix**分支，而需要多少个**hotfix**分支取决在**可能的hotfix阶段**我们有多少个已经发布的，而且正在支持的版本。

举个例子，如果我的产品是持续发布的，比如发布周期是1天两次，那么这个**产品支持阶段**就是半天，而我们大部分hotfix开发测试的时间也得差不多半天了吧，所以对于这种持续发布的产品，**hotfix**就不需要了。

## Release分支

对于**release**分支，也有几个概念。第一个是**版本稳定时间**，比如我们要上线一个版本，需要做一系列的测试才有足够的信心认为这个版本已经足够稳定可以发布了，这个时间就是**版本稳定时间**，他基本上就是在上线之前我们需要的手动测试或者自动化测试要花费的时间。第二个概念是**代码冻结日期**，**代码冻结日期**=**产品发布日期**-**版本稳定时间**。只有在**版本稳定时间**比较长的情况下才需要**release**分支，因为**release**分支存在的意义就是让我们可以在这个**release**分支冻结代码，然后在这个分支上做发布前的测试。

举个例子，如果我们的产品有覆盖率非常全的自动化测试，而且这个自动化测试只需要5分钟就能跑完。那么我们的**版本稳定时间**就是5分钟，那么我们就不需要有**代码冻结日期**，也不需要有**release**分支，因为我们只需要5分钟的时间就知道这个产品能不能发布了。

# GitHub Flow

从上面的分析我们可以看到，如果一个产品是持续发布的，其实就没有**hotfix**和**release**分支存在的意义了，同时，因为是持续发布，所以**develop**分支和**master**分支差别不大，那么也就没有必要维护这两个主分支了，这其实就是我们下面要说的**GitHub Flow**。

**GitHub Flow**最早是在2011年的这篇文章[GitHub Flow](http://scottchacon.com/2011/08/31/github-flow.html)中提出来的，如下图所示。

{% limg githubflow1.png %}

**GitHub Flow**有如下要点：
1. **master**是部署分支
1. 所有针对下一个发布的开发都在**feature**分支
1. **feature**分支经过测试之后merge回**master**分支，然后发布
1. 如果发布之后发现有bug，回滚到**master**分支的上一个发布版本

这个分支模型在**feature**分支merge回**master**分支时会有潜在的冲突，这就要求每次在**feature**分支开始做发布前的测试的时候先从**master**分支拉代码，并且在这个**feature**分支完成发布前的测试工作时，其他**feature**分支不能开始发布前的测试工作（因为不然的话这个测试是无效的，因为随后**master**分支会有新的代码需要拉下来）。

另外这个分支模型有一个缺点就是**master**分支上不是都是真正可发布的，因为包含了一些发布过后发现有问题然后回滚到上一个发布的情况。所以之后**GitHub Flow**做了一次修正，可以参见现在的[GitHub Flow](https://guides.github.com/introduction/flow/)，如下图所示。

{% limg githubflow2.png %}

它和2011年那篇文章的区别在于，发布直接发生在**feature**分支上，等这个发布没有问题才会merge回**master**分支。这样**master**上就全部是真正可以发布的版本了。但是这个分支模型的另一个风险是有可能从一个**feature**分支发布了一个版本之后忘记merge回**master**分支了。

可以看到这种**GitHub Flow**分支模型和**Git Flow**相比特别简单，非常适合持续发布的产品。全部的开发都通过**feature**分支来做，下面我们来看看**feature**分支。

## Feature分支

在Martin Fowler的[Feature Branch](https://www.martinfowler.com/bliki/FeatureBranch.html)里讨论过对**feature**分支的看法，最大的问题在于如果多个**feature**分支长期并存，可能会带来之后merge时有很多conflict要处理，而且如果**feature**分支不经常merge到主分支，其实是和持续集成违背的。

在我看来，**feature**分支只有在一种情况下才是必须存在的，那就是这个功能的**开发时间比下次产品发布的时间还要长**。这个对于持续发布的产品来说当然是成立的，比如使用**GitHub Flow**的GitHub，每天发布几次，所以使用**feature**分支是合适的，而且他们的**feature**分支也不会存在太长时间，所以merge时的冲突也不是很严重。还有一种常见的情况是开源软件的开发，每个开发人员并不能保证功能开发的进度，比如可能每周才工作几个小时，这种情况下就适合使用**feature**分支了。

# Trunk Based Development

下面介绍另外一种分支模型**基于主干的开发[Trunk Based Development](https://trunkbaseddevelopment.com/)**，如下图所示。

{% limg trunkbaseddevelopment.png %}

**Trunk Based Development**的要点是：
1. 团队只提交代码到**Trunk**（也就是**master**）。
1. **Trunk**（也就是**master**）并不是一直可以发布的。

## 主干发布
**Trunk Based Development**有两种发布策略，一种是**[主干发布（release from trunk）](https://trunkbaseddevelopment.com/release-from-trunk/)**，直接基于主干上的代码进行测试，然后通过测试后发布。这种发布策略通常适用于发布周期很短的产品。如下图所示。

{% limg trunkbaseddevelopment_releasefromtrunk.png %}

如果发布之后发现bug，可能直接从**Trunk**发布一个新的版本。如果产品有固定的发布周期现在还没到，也可以这个时候从上次发布的那个点做一个**hotfix**分支，然后在这个分支上部署hotfix，如下图所示。

{% limg trunkbaseddevelopment_releasefromtrunk_hotfix.png %}

需要注意一点的是**Trunk Based Development**遵循**upstream first policy**，就是说必须在**master**分支上重现这个bug，然后在**master**分支上fix这个bug，测试过之后再把这个fix通过cherry-pick到**hotfix**分支。这样有如下的好处：
1. 保证所有的bug fix都在trunk里
1. 因为一直在最新的代码（而不是当时发布的代码）上做修改，降低了引入regression的可能性。

## release分支发布

**Trunk Based Development**的第二种发布方式是**release分支发布**，如下图所示。

{% limg trunkbaseddevelopment_branchforrelease.png %}

这种发布模式就是在需要**代码冻结（code freeze）**的时候拉出来一个**release**分支，然后测试这个分支，如果遇到bug，也像上面说的一样遵循**upstream first policy**，就是说必须在**master**分支上重现这个bug，然后在**master**分支上fix这个bug，测试过之后再把这个fix通过cherry-pick到**release**分支。等产品发布之后如果遇到bug，就把这个**release**分支当成**hotfix**分支来发布hotfix，和上面一样了。

## 关于cherry-pick

有很多关于cherry-pick的讨论，比如[Stop cherry-picking, start merging](https://devblogs.microsoft.com/oldnewthing/20180323-01/?p=98325)这里有一系列的讨论，有兴趣的可以看看。

# 后记

上面我简单介绍了一下这3种常见的分支模型，分别是**Git Flow**， **GitHub Flow**和**Trunk Based Development**。其实就像我之前说的，更重要是理解产品发布的策略，基于团队本身开发和测试的能力来选择一种分支策略。

总的来说，分支策略越简单团队的额外开销会越少，从而团队的效率会越高。如何达成简单的分支策略，很大一部分取决于产品的**发布周期**和**版本稳定时间**，归根到底取决于在发布一个产品之前我们需要花多长时间来测试。所以在我看来，**提高团队的自动化测试能力是最根本的**。
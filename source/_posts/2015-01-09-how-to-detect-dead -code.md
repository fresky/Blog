title: 查找无用代码Dead Code的一些心得
date: 2015-01-09 16:55:02
categories:
tags: Refactoring
description: 把这些没用的代码删除对提高代码质量，降低维护成本都有很大的好处。本文就简单总结一下我认为有效的可以找到无用代码的一些心得：1. 使用工具帮助查找无用代码。2. 通过测试覆盖率Code Coverage寻找潜在的无用代码。3. 查找重复的代码。4. 阅读代码，理解代码。
---
一个系统开发久了难免会有一些没用的代码Dead Code。把这些没用的代码删除对提高代码质量，降低维护成本都有很大的好处。本文就简单总结一下我认为有效的可以找到无用代码的一些心得。

## 使用工具帮助查找无用代码
###Resharper——收费，只适用于C#。

[Resharper](http://www.jetbrains.com/resharper/)是一个C#开发必备神器了。它能检查出很多代码问题，包含一些无用代码，参见下图。

{% limg resharperdeadcode.png %}

###Visual Studio的Code Analysis (FXCop)——集成在Visual Studio中，只适用于C#。

[Code Analysis for Managed Code Warnings](http://msdn.microsoft.com/en-us/library/ee1hzekz.aspx)可以针对C#做很多检查，其中和Dead Code相关的检查有如下几个：

1. [CA1801: Review unused parameters](http://msdn.microsoft.com/en-us/library/ms182268.aspx)  
1. [CA1804: Remove unused locals](http://msdn.microsoft.com/en-us/library/ms182278.aspx)  
1. [CA1811: Avoid uncalled private code](http://msdn.microsoft.com/en-us/library/ms182264.aspx)  
1. [CA1812: Avoid uninstantiated internal classes](http://msdn.microsoft.com/en-us/library/ms182265.aspx)  
1. [CA1823: Avoid unused private fields](http://msdn.microsoft.com/en-us/library/ms245042.aspx)  

可以在Visual Studio中自己创建一个RuleSet，只包含这几个规章，然后跑一遍就行了，参见下图。

{% limg fxcop1.png %}
{% limg fxcop2.png %}
{% limg fxcop3.png %}

## 通过测试覆盖率Code Coverage寻找潜在的无用代码

如果一段代码的测试覆盖率是0，那它有可能就是Dead Code。当然这条规则只在测试覆盖率很高，而且测试质量很高的情况下才有效。这个测试覆盖率可以是单元测试加上集成测试，再加上UI自动化测试。测试质量越高，越能容易的发现潜在的Dead Code。所以如果严格按照TDD的流程进行开发，理论上就不应该有无用代码，当然前提是驱动开发的那个测试是有用的：）。

## 查找重复的代码
我认为重复代码也是一种无用代码，而且可能比无用代码危害还大。查找重复代码相对来说比查找无用代码容易一些，也有很多的工具可以使用，比如：
### Visual Studio —— 只适用于C#。
我之前的博客[VS2012很cool的新功能：检查重复代码](/2012/08/21/find-cloned-code-in-csharp-with-visual-studio-2012/)介绍过，参加下图。

{% limg vscodeclone.png %}

### Simian —— 非商用免费，商用收费
[Simian](http://www.harukizaemon.com/simian/index.html)支持Java, C#, C, C++, COBOL, Ruby, JSP, ASP, HTML, XML, Visual Basic, Groovy。

## 阅读代码，理解代码
这个不用说了，什么工具都比不上熟悉代码的你：）

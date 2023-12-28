title: Visual C++中的编译器优化
date: 2015-07-13 12:48:00
categories:
tags: Programming
description:
---
[Compilers - What Every Programmer Should Know About Compiler Optimizations](https://msdn.microsoft.com/en-us/magazine/dn904673.aspx)和[Part 2](https://msdn.microsoft.com/en-us/magazine/dn973015.aspx)介绍了Visual C++中的编译器优化。摘录几个定义如下：

链接期代码生成（Link-Time Code Generation）：C++编译器本来是编译单个文件，所以优化只能基于单个源文件。但是如果打开这个LTCG之后，编译会生成C中间语音（C Inter­mediate Language），然后链接器会做全程序优化（Whole Program Optimization）。Visual Studio的Release配置就是这个LTCG。

循环优化（Loop Optimization）：主要包括1）循环展开，2）自动向量化，3）循环不变量代码移动。

寄存器分配优化（Register Allocation）。x86处理器有8个32位通用寄存器，8个80位的浮点数寄存器，8个128位向量寄存器。64位处理器有16个64位通用寄存器，8个80位浮点数寄存器和至少16个向量寄存器（每个至少128位）。因为寄存器没有地址，所以如果一个变量需要取地址，不会放在寄存器里。

指令调度优化（Instruction Scheduling）。基于编译的指令重排（instruction reordering），基于硬件的乱序执行（out-of-order）。
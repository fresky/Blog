title: NASA关于如何写出安全代码的10条军规
date: 2015-05-31 15:23:18
categories: 
tags: Development
description:
---
[The Power of Ten – Rules for Developing Safety Critical Code](http://pixelscommander.com/wp-content/uploads/2014/12/P10.pdf)列出了NASA关于如何写出安全代码的10条规则。

1. 所有的控制流程非常简单，不能有GOTO，不能有直接或间接的递归。  
1. 所有的循环都必须有一个固定的上届。比如有一个非常简单的静态检查工具可以证明这个上届不会被超过。  
1. 在初始化之后不能再用动态内存分配。  
1. 所有函数都必须可以在1页纸打印出来，就是说不能超过60行。  
1. 每个函数平均要有2个Assert，Assert都不能有副作用，并且定义为布尔值。  
1. 数据对象的作用域要定义的尽量小。  
1. 每个函数调用别的非void函数时，必须检查返回值。每个函数内部必须做参数验证。  
1. 预处理只能用来包含头文件和简单的宏定义。  
1. 指针的使用要严格限制。不能超过一层的dereference。不能用函数指针。  
1. 所有代码都需要能编译通过，没有警告。每天都要运行最先进的静态检查工具，不能有任何警告。  


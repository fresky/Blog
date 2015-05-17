title: C#性能优化的一些技巧
date: 2015-05-17 23:22:22
categories:
tags: CSharp
description: 
---
[.NET Perls: Optimization](http://www.dotnetperls.com/optimization)列举了一些关于C#如何做性能优化的技巧和示例，我挑了几个列在下面，关于范例请参看原文。要注意的是这里提到的一些做法会导致程序可读性和可维护性变差，所以一定要在必要的时候再使用，对大部分非hot path的程序来说，可读性和可维护性通常比性能更重要。

1. 静态方法，静态成员。非inline的实例方法总是比非inline的静态方法慢。  
1. 减少函数参数。函数的参数会在函数被调用时复制。  
1. 使用常量。常量的值会直接放到指令里。  
1. inline。.NET 4.5可以用`AggressiveInlining`来告诉编译器尽量inline。  
1. `switch`。`switch`通常比`if`快。  
1. 扁平化数组。一维数组比二维数组快，可以自己计算下标来用一维数组模拟二维数组。  
1. `StringBuilder`。如果要把一些字符串连起来，`StringBuilder`通常会快一些。  
1. 字符数组。通常使用字符数组比使用字符串快。  
1. Byte数组。如果只需要存储ASCII字符，Byte数组会节省空间。  
1. 数组。直接操作数组通常会比其他集合快。  
1. 容量。初始化集合是给一个容量会减少扩展集合的开销。  
1. 避免把`Struct`作为函数参数。  
1. 使用查找表。  
1. 针对`int`做字符串表示的缓存。如果程序需要经常调用`int`的`ToString`，这么做会节省很多时间。  
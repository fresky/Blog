title: C#和C++中的float类型
date: 2015-05-24 23:43:11
categories:
tags: Programming
description: 
---
# 浮点数介绍

[浮点数（Floating Point）](http://en.wikipedia.org/wiki/Floating_point)是计算机中用二进制表示实数的一种近似表示方式。就是用科学记数法来表示一个实数，分为3个部分，第一个部分是符号位，第二个部分是指数，第三个部分是尾数。

[IEEE 754](http://en.wikipedia.org/wiki/IEEE_floating_point)是IEEE的计算机表示浮点数的标准。它包含下面几种：

|Type|Sign|Exponent|Significand field|Total bits|Exponent bias|Bits precision|Number of decimal digits|
|----|----|----|----|----|----|----|----|
|Half (IEEE 754-2008)|1|5|10|16|	15|11|~3.3|
|Single|1|8|23|32|127|24|~7.2|
|Double|1|11|52|64|1023|53|~15.9|
|x86 extended precision|1|15|64|80|	16383|64|~19.2|
|Quad|1|15|112|128|16383|113|~34.0|

最常见的就是单精度（Single）和双精度（Double），就是C++和C#中的float和double。

先举个用float来表示实数的例子，比如我们要表示`100`，换算成2进制就是`1100100`，那么就是`1.1001*10^6`。所以符号位是0，指数是6加上Exponent bias（127）是`133`，换算成2进制是`10000101`，尾数是`1001`。那么拼一起就是`0 10000101 10010000000000000000000`。

# 浮点数中的特殊值

接着我们看看根据定义，float中那些特殊值都是啥。看如下的C++程序，把2进制的表示转换成float数值并且打印出来。

```cpp
int   binaryPresentation;
float floatPresentation;

binaryPresentation = 0x42C80000; // 0 10000101 10010000000000000000000
floatPresentation = *(float *)&binaryPresentation;
printf("100: %g\n", floatPresentation);

binaryPresentation = 0x00000000; // 0 00000000 00000000000000000000000
floatPresentation = *(float *)&binaryPresentation;
printf("Positive ZERO: %g\n", floatPresentation);

binaryPresentation = 0x80000000; // 1 00000000 00000000000000000000000
floatPresentation = *(float *)&binaryPresentation;
printf("Negative ZERO: %g\n", floatPresentation);

binaryPresentation = 0x00000001; // 0 00000000 00000000000000000000001
floatPresentation = *(float *)&binaryPresentation;
printf("Positive Min Denorm Float: %g\n", floatPresentation);

binaryPresentation = 0x007fffff; // 0 00000000 11111111111111111111111
floatPresentation = *(float *)&binaryPresentation;
printf("Positive Max Denorm Float: %g\n", floatPresentation);

binaryPresentation = 0x00800001; // 0 00000001 00000000000000000000000
floatPresentation = *(float *)&binaryPresentation;
printf("Positive Min Normalize float: %g\n", floatPresentation);

binaryPresentation = 0x7f7fffff; // 0 11111110 11111111111111111111111
floatPresentation = *(float *)&binaryPresentation;
printf("Positive Max Normalize float: %g\n", floatPresentation);

binaryPresentation = 0x7F800000; // 0 11111111 00000000000000000000000 
floatPresentation = *(float *)&binaryPresentation;
printf("Positive infinite: %g\n", floatPresentation);

binaryPresentation = 0xFF800000; // 1 11111111 00000000000000000000000 
floatPresentation = *(float *)&binaryPresentation;
printf("Negative infinite: %g\n", floatPresentation);
```

输出如下：

```
100: 100
Positive ZERO: 0
Negative ZERO: -0
Positive Min Denorm Float: 1.4013e-045
Positive Max Denorm Float: 1.17549e-038
Positive Min Normalize float: 1.17549e-038
Positive Max Normalize float: 3.40282e+038
Positive infinite: 1.#INF
Negative infinite: -1.#INF
```

# C++中的浮点数特殊值

接着看看C++中的浮点数特殊值是怎么定义的：

```cpp
printf("denorm_min: %g\n", std::numeric_limits<float>::denorm_min());
printf("epsilon: %g\n", std::numeric_limits<float>::epsilon());
printf("infinity: %g\n", std::numeric_limits<float>::infinity());
printf("lowest: %g\n", std::numeric_limits<float>::lowest());
printf("max: %g\n", std::numeric_limits<float>::max());
printf("min: %g\n", std::numeric_limits<float>::min());
printf("quiet_NaN: %g\n", std::numeric_limits<float>::quiet_NaN());
printf("signaling_NaN: %g\n", std::numeric_limits<float>::signaling_NaN());
printf("round_error: %g\n", std::numeric_limits<float>::round_error());
```

输出如下：
```
denorm_min: 1.4013e-045
epsilon: 1.19209e-007
infinity: 1.#INF
lowest: -3.40282e+038
max: 3.40282e+038
min: 1.17549e-038
quiet_NaN: 1.#QNAN
signaling_NaN: 1.#QNAN
round_error: 0.5
```

# CSharp中的浮点数特殊值

再来看看C#中的浮点数特殊值是怎么定义的：

```csharp
Console.WriteLine("Epsilon:" + float.Epsilon);
Console.WriteLine("MaxValue:" + float.MaxValue);
Console.WriteLine("MinValue:" + float.MinValue);
Console.WriteLine("NaN:" + float.NaN);
Console.WriteLine("PositiveInfinity:" + float.PositiveInfinity);
Console.WriteLine("NegativeInfinity:" + float.NegativeInfinity);
```

输出如下：
```
Epsilon:1.401298E-45
MaxValue:3.402823E+38
MinValue:-3.402823E+38
NaN:NaN
PositiveInfinity:Infinity
NegativeInfinity:-Infinity
```

可以看出来C#和C++中关于Epsilon和MinValue的定义是不一样的。

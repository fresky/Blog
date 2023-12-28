---
layout: post
title: "C++的C4305和C4800的编译警告"
date: 2013-05-24
comments: true
tags: Programming
---
<p>今天在改代码时发现了这两个warning，<a href="http://msdn.microsoft.com/en-us/library/0as1ke3f%28v=vs.80%29.aspx">C4305</a> - 'identifier' : truncation from 'type1' to 'type2' 和<a href="http://msdn.microsoft.com/en-us/library/b6801kcy%28v=VS.71%29.aspx">C4800</a> - 'type' : forcing value to bool 'true' or 'false' (performance warning)。</p>  <p>大家看看下面的代码吧，一目了然。：）</p>  

```cpp
    int a1 = 1;
    int a2 = 2;
    MyEnum m = MyEnum::a;

    bool b = 1;             // no warning
    b = 2;                  // warning C4305: '=' : truncation from 'int' to 'bool'
    b = a1;                 // warning C4800: 'int' : forcing value to bool 'true' or 'false' (performance warning)    
    b = a2;                 // warning C4800: 'int' : forcing value to bool 'true' or 'false' (performance warning)    
    b = MyEnum::a;          // no warning
    b = m;                  // warning C4800: 'MyEnum' : forcing value to bool 'true' or 'false' (performance warning)    
    b = !!a2;               // no warning
    b = a2!=0;              // no warning
```
---
layout: post
title: "C++中怎么阻止一个类被继承"
date: 2012-12-04
comments: true
tags: CPP
---
<p>C#中可以用sealed关键字，但是，C++中怎么阻止一个类被继承呢？</p>  <p>方法就是把这个类的构造函数声明成private的，这样就不能被继承了。当然更好的办法是用非技术的手段了：）</p>  <p>方法1： 构造函数private，提供一个Factory方法，缺点就是使用者必须用这个factory方法，不能直接使用这个类。</p>  
```cpp
class NoDerive {
    NoDerive(){};
public:
    static NoDerive GetNoDerive(){ return NoDerive();};
    ~NoDerive(){};
};
```

<p>方法2： 构造函数public，但是虚拟继承自一个private构造函数，没有方法1的缺点，但是看着有点hack。</p>

```cpp
class NoDerive;
class NoDeriveBase
{
    friend class NoDerive;
    NoDeriveBase(){};
};
class NoDerive:private virtual NoDeriveBase {
public:
    NoDerive(){};
    ~NoDerive(){};
};
```
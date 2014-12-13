---
layout: post
title: "C++，想要提高性能，那就值传递（pass by value）吧。"
date: 2012-12-11
comments: true
tags: CPP
---
<p>通常我们在学习写C++程序的时候都听过这样的说法，作为函数的参数，应该引用传递pass by const refercence，这样不会有值传递引起拷贝问题，可以提高性能，但是<a href="http://cpp-next.com/archive/2009/08/want-speed-pass-by-value/">Want Speed? Pass by Value</a>这篇文章的标题就是想要提高性能吗？那就值传递吧。</p>
<p>这篇文章讲了右值rvalue和返回值优化RVO，然后得出了原则：</p>
<h3>不要复制函数的参数。应该通过值传递的方式让编译器来复制。</h3>
<p>其实这并不是要颠覆我们以前说的值传递和引用传递的取舍，而是说，如果在我们的函数里面需要拷贝一份参数的话，那就不要通过传递引用，然后函数内部在调用拷贝构造的方式。而是应该直接用值传递的方式，这样编译器会有更大的空间来进行优化，提高性能。</p>
<p>贴两个文章中举得例子吧。</p>
<h2>1. 对vector排序。</h2>
<p>下面的代码性能不好，因为显示的拷贝不能被编译器优化。</p>

```cpp
std::vector<std::string> 
sorted(std::vector<std::string> const& names) // names passed by reference
{
    std::vector<std::string> r(names);        // and explicitly copied
    std::sort(r);
    return r;
}
```
<p>下面的代码性能好，因为编译器可以用RVO优化，可以避免拷贝。而且就算编译器没有优化，最坏也就是和上面的代码一样，而且还少了一行，看起来更清楚一些。</p>

```
std::vector<std::string> 
sorted(std::vector<std::string> names)
{
    std::sort(names);
    return names;
}
```
<h2>2.赋值运算符。（其实就是Copy-And-Swap-Idiom）</h2>
<p>性能不好：</p>

```
T& T::operator=(T const& x) // x is a reference to the source
{ 
    T tmp(x);          // copy construction of tmp does the hard work
    swap(*this, tmp);  // trade our resources for tmp's
    return *this;      // our (old) resources get destroyed with tmp 
}
```
<p>性能好：</p>

```
T& operator=(T x)    // x is a copy of the source; hard work already done
{
    swap(*this, x);  // trade our resources for x's
    return *this;    // our (old) resources get destroyed with x
}
```
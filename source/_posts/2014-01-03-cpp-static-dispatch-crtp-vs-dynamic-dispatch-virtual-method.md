---
layout: post
title: "C++的静态分发(CRTP)和动态分发(虚函数多态)的比较"
date: 2014-01-03 18:27
comments: true
tags: Programming
---

虚函数是C++实现多态的工具，在运行时根据虚表决定调用合适的函数。这被称作动态分发。虚函数很好的实现了多态的要求，但是在运行时引入了一些开销，包括：  
1. 对每一个虚函数的调用都需要额外的指针寻址  
2. 虚函数通常不能被inline，当虚函数都是小函数时会有比较大的性能损失  
3. 每个对象都需要有一个额外的指针指向虚表  

所以如果是一个对性能要求非常严格的场合，我们就需要用别的方式来实现分发，这就是今天这篇博客的主角[CRTP](http://en.wikipedia.org/wiki/Curiously_recurring_template_pattern)。

CRTP通过模板实现了静态分发，会带来很多性能的好处。可以参见[The cost of dynamic (virtual calls) vs. static (CRTP) dispatch in C++](http://eli.thegreenplace.net/2013/12/05/the-cost-of-dynamic-virtual-calls-vs-static-crtp-dispatch-in-c/)看一下性能比较。

下面简单介绍一下怎么实现CRTP。

首先看我们的父类：
```cpp
template<typename Derived>  class Parent 
{
public:
    void SayHi()
    {
        static_cast<Derived*>(this)->SayHiImpl();
    }
private:
    void SayHiImpl() // no need if no need the default implementation
    {
        cout << "hi, i'm default!" << endl;
    }
};
```

它是一个模板类，它有一个需要接口函数是```SayHi```。它有一个默认实现在```SayHiImpl```中。

再来看看它的子类：

```cpp
class ChildA :public Parent<ChildA>
{
public:
    void SayHiImpl()
    {
        cout << "hi, i'm child A!" << endl;
    }
};

class ChildB :public Parent<ChildB>
{
};
```

我们可以看到```ChildA```和```ChildB```继承自这个模板类，同时```ChildA```有自己的实现。

在写一个普通的用虚函数实现分发的类：

```cpp
class ParentB
{
public:
    void virtual SayHi()
    {
        cout << "hi, i'm default!" << endl;
    }
};

class ChildC : public ParentB
{
public:
    void SayHi()
    {
        cout << "hi, i'm ChildC!" << endl;
    }
};

class ChildD : public ParentB
{
}; 
```

然后是调用这两个父类的函数：


```cpp
template<typename Derived> void CRTP(Parent<Derived>& p)
{
    p.SayHi();
}

void Dynamic(ParentB& p)
{
    p.SayHi();
}
```

再来看看```main```函数：

```cpp
int _tmain(int argc, TCHAR* argv[])
{

    ChildA a;
    CRTP(a);
    cout << "size of ChildA: " << sizeof(a) << endl;

    ChildB b;
    CRTP(b);
    cout << "size of ChildB: " << sizeof(b) << endl;

    ChildC c;
    Dynamic(c);
    cout << "size of ChildC: " << sizeof(c) << endl;

    ChildD d;
    Dynamic(d);
    cout << "size of ChildD: " << sizeof(d) << endl;

    return 0;
}
```

如果运行这个程序，可以看到如下的输出，可以看到CRTP可以实现和虚函数一样的功能，但是内存大小会有很大优势，关于对象内存可以参见我之前的博客[怎么看C++对象的内存结构 和 怎么解密C++的name Mangling](/2012/12/23/blogpost/)：

```
hi, i'm child A!
size of ChildA: 1
hi, i'm default!
size of ChildB: 1
hi, i'm ChildC!
size of ChildC: 4
hi, i'm default!
size of ChildD: 4
Press any key to continue . . .
```

完整代码参见[gist](https://gist.github.com/fresky/8237006)。
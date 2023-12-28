---
layout: post
title: "用CRTP在C++中实现静态函数的多态"
date: 2014-01-06 10:50
comments: true
tags: Programming
---

我上一篇博客[C++的静态分发(CRTP)和动态分发(虚函数多态)的比较](/2014/01/03/cpp-static-dispatch-crtp-vs-dynamic-dispatch-virtual-method/)介绍了如何用CRTP(Curiously Recurring Template Pattern)实现静态分发，今天再讲另外一个CRTP的例子。在C++中静态函数是不能设成virtual的，但是用CRTP可以实现静态函数的多态。

先看代码:

```cpp
template<typename Derived>  class Parent 
{
public:
    void SayHi()
    {
        static_cast<Derived*>(this)->SayHiImpl();
    }
    void static SayHiStatic()
    {
        Derived::SayHiStatic();
    }
private:
    void SayHiImpl()
    {
        cout << "hi, i'm default!" << endl;
    }
};

class ChildA :public Parent<ChildA>
{
public:
    void SayHiImpl()
    {
        cout << "hi, i'm child A!" << endl;
    }
    void static SayHiStatic()
    {
        cout << "Static hi from Child A" << endl;
    }
};

class ChildB :public Parent<ChildB>
{
public:
    void static SayHiStatic()
    {
        cout << "Static hi from Child B" << endl;
    }
};
```

和上次的代码类似，这次父类有一个静态的interface，两个子类分别实现了自己的静态实现。

再看调用静态```interface```的函数和```main```函数。

```
template<typename Derived> void CRTP(Parent<Derived>& p)
{
    p.SayHiStatic();
}

int _tmain(int argc, TCHAR* argv[])
{
    ChildA a;
    CRTP(a);
    cout << "size of ChildA: " << sizeof(a) << endl;

    ChildB b;
    CRTP(b);
    cout << "size of ChildB: " << sizeof(b) << endl;

    return 0;
}
```

运行可以得到如下输出：

```
Static hi from Child A
size of ChildA: 1
Static hi from Child B
size of ChildB: 1
```
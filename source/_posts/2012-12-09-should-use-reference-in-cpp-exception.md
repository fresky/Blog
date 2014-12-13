---
layout: post
title: "C++应该用引用来捕捉异常"
date: 2012-12-09
comments: true
tags: CPP
---
<p>在C++中catch异常时的参数应该用引用，主要原因还是对象，引用，指针的构造析构原理。下面用代码实例解释一下原因。</p>  <p>先来看我们定义了两个异常，SubException继承BaseException，有一个虚函数打印信息。</p>  

```cpp
class BaseException
{
public:
    BaseException(){ cout<<"BaseExeption"<<endl; };
    BaseException(BaseException& ){cout<<"BaseExeption Copy from BaseException"<<endl;};
    BaseException(SubException& ){cout<<"BaseExeption Copy from SubException"<<endl;};
    ~BaseException(){cout<<"~BaseExeption"<<endl;};
    virtual void PrintMessage(){cout<<"I'm BaseException"<<endl;}
};

class SubException:public BaseException
{
public:
    SubException(){cout<<"SubException"<<endl;}
    SubException(SubException& ):BaseException(){cout<<"SubException Copy from SubException"<<endl;};
    ~SubException(){cout<<"~SubException"<<endl;}
    virtual void PrintMessage(){cout<<"I'm SubException"<<endl;}
};
```

<h3>1. Catch对象。</h3>

<p>再来看用对象做为参数catch的例子。</p>

```cpp
void CatchBaseObject(bool throwbase)
{
    cout<<"==================="<<endl<<"CatchBaseObject "<<(throwbase?"ThrowBase":"ThrowSub")<<endl;
    try
    {
        throwbase? throw BaseException(): throw SubException();
    }
    catch (BaseException e)
    {
        e.PrintMessage();
    }
}
```

<p>输出结果如下：</p>

```
===================
CatchBaseObject ThrowBase
BaseExeption
BaseExeption Copy from BaseException
I'm BaseException
~BaseExeption
~BaseExeption
===================
CatchBaseObject ThrowSub
BaseExeption
SubException
BaseExeption Copy from BaseException
I'm BaseException
~BaseExeption
~SubException
~BaseExeption
```

<p>从这个输出中我们可以看出问题：</p>

<ol>
  <li>有2个对象被构建出来。</li>

  <li>catch BaseException时发生了slicing，sub的信息丢掉了。</li>
</ol>

<p>除了这些问题之外，如果catch住在throw的话，要注意只能用throw，而不能用throw e，那样会再次生成临时对象，并且丢失原来的sub信息。测试代码如下：</p>

```cpp
void CatchBaseObjectForThrowSubAndThrowItAgain()
{
    cout<<"==================="<<endl<<"CatchBaseObjectForThrowSubAndThrowItAgain "<<endl;
    try
    {
        try
        {
            throw SubException();
        }
        catch (BaseException e)
        {
            e.PrintMessage();
            throw e;
        }
    }
    catch (SubException e)
    {
        e.PrintMessage();
    }
}
```

<p>输出如下：</p>

```
===================
CatchBaseObjectForThrowSubAndThrowItAgain
BaseExeption
SubException
BaseExeption Copy from BaseException
I'm BaseException
BaseExeption Copy from BaseException
Press any key to continue . . .
```

<p>再次throw时已经变成了一BaseException，所以第二个catch不能抓住，导致程序终止。</p>

<h3>2. Catch指针。</h3>

<p>如果在throw exception的地方用的栈上的对象的地址，像如下代码所示：</p>

```cpp
void CatchBasePointer(bool throwbase)
{
    cout<<"==================="<<endl<<"CatchBasePointer "<<(throwbase?"ThrowBase":"ThrowSub")<<endl;
    try
    {
        throwbase? throw &BaseException(): throw &SubException();
    }
    catch (BaseException* e)
    {
        e->PrintMessage();
    }
}
```

<p>得到输出如下：</p>

```
===================
CatchBasePointer ThrowBase
BaseExeption
~BaseExeption
I'm BaseException
===================
CatchBasePointer ThrowSub
BaseExeption
SubException
~SubException
~BaseExeption
I'm BaseException
```

<p>我们可以看到如下问题：</p>

<ol>
  <li>生成的那个exception时临时对象，除了try块就被析构了。</li>

  <li>catch住的异常对象发生了slicing。</li>
</ol>

<p>为了保证不被析构，我们得new在堆上或者用static，如下代码展示用static：</p>

```cpp
void CatchBasePointerThrowStaticSub()
{
    cout<<"==================="<<endl<<"CatchBasePointerThrowStaticSub "<<endl;
    try
    {
        static SubException s;
        throw &s;
    }
    catch (BaseException* e)
    {
        e->PrintMessage();
    }
}
```

<p>得到输入如下：</p>

```
===================
CatchBasePointerThrowStaticSub
BaseExeption
SubException
I'm SubException
```

<p>这时这个对象还在，因为用的是指针，所以也还有多态，但是要不要delete就不好办了，如果是static的，不需要delete，但是如果是new出来的，需要delete。</p>

<h3>3.Catch引用。</h3>

<p>如果换成引用，输出如下：</p>

```
===================
CatchBaseRef ThrowBase
BaseExeption
I'm BaseException
~BaseExeption
===================
CatchBaseRef ThrowSub
BaseExeption
SubException
I'm SubException
~SubException
~BaseExeption
```

我们可以看到，没有多余的临时对象被创建出来，而且保持了多态，是我们期望的行为。

<p>所以永远要用引用的方式来捕捉异常。本文例子的完整代码在<a href="https://github.com/fresky/CppExample">github</a>上。</p>
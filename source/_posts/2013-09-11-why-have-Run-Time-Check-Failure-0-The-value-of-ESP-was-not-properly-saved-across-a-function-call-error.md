---
layout: post
title: "C++程序在debug模式下遇到Run-Time Check Failure #0 - The value of ESP was not properly saved across a function call问题"
date: 2013-09-11
comments: true
tags: Programming
---
<p>今天遇到一个Access Violation的crash，只看crash call stack没有找到更多的线索，于是在debug模式下又跑了一遍，遇到了如下的一个debug的错误提示框：</p>  <p><a href="http://images.cnitblog.com/blog/163228/201309/11150255-8d44d6fb8f3942dc925c47a4f4291b2b.png"><img title="runtimecheckfailureESP" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="runtimecheckfailureESP" src="http://images.cnitblog.com/blog/163228/201309/11150257-27be9445885646ac853bf1e6c3ea0861.png" width="411" height="308" /></a></p>  <p>这个是什么原因呢？我们来看一个简单的例子来重现这个错误。</p>  <p>假设我们有2个父类，分别是BaseA和BaseB。有一个子类Child，继承自BaseA和BaseB。</p>

```c++
class BaseA
{
public:
  virtual void foo()=0;
};

class BaseB
{
public:
  virtual void bar(int a)=0;
};

class Child: public BaseA, public BaseB
{
public:
  void foo()
  {
    cout<<"i'm foo in Child!"<<endl;
  };
  void bar(int a)
  {
    cout<<"i'm bar in Child!"<<endl;
  };
};
```

<p>假设我们有如下的main函数：</p>

```c++
int main() {

  BaseB* b = new Child();
  BaseA* a = (BaseA*)b;
  BaseA* a2 = dynamic_cast<BaseA*> (b);
  // This is actually calling bar(),
  // and will cause Runtime check failure about ESP if the foo and bar have different signature.
  a->foo();
  a2->foo();
}
```

<p>在这个main函数里a是通过C-Style的强转转来的，a2是通过dynamic_cast转换来的。如果我们运行这个程序，在release下就会报access violation的crash，在debug下就会出上面列的ESP错误的对话框。</p>

<p>函数的输出如下图所示：</p>

<p><a href="http://images.cnitblog.com/blog/163228/201309/11150258-615ef9ad31c64a598c35bb7adc170bde.png"><img title="espoutput" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="espoutput" src="http://images.cnitblog.com/blog/163228/201309/11150300-fbca0f8bad97479ab07bd8d0d53b6e08.png" width="244" height="92" /></a></p>

<p>这就能解释为什么会出这个ESP的错误了，因为我们想调用foo，但是实际上调用的bar，但是foo和bar的函数签名不一样，所以导致了出现那个debug版本下的ESP的错误提示。但是加入我们把bar（int a）中的参数去掉，这样foo和bar就有一样的函数签名了，这样及时在debug版本下也不会出这个ESP的提示，在release或者debug模式下都不会报任何错误，但是我们在runtime调用了错误的函数！！！</p>

<p>可以看看下图回忆一下vtable是怎么回事：）</p>

<p><a href="http://images.cnitblog.com/blog/163228/201309/11150301-4e8c130427464fc7ba196fb2f84f13c6.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://images.cnitblog.com/blog/163228/201309/11150303-2008ada2678c49a5a1c1052b06dc916f.png" width="621" height="253" /></a></p>

<p>&#160;</p>

也可以参考[怎么看C++对象的内存结构 和 怎么解密C++的name mangling](/2012/12/23/how-to-dump-cpp-objects-memory-layout-and-crack-name-mangling/)来看看C++对象的内存结构。

<p>对于ESP的错误，可以参考stackoverflow上的这个问题：<a href="http://stackoverflow.com/questions/8626160/run-time-check-failure-0-the-value-of-esp-was-not-properly-saved-across-a-fun">c++ - Run-Time Check Failure #0 - The value of ESP was not properly saved across a function call - Stack Overflow</a></p>

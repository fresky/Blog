---
layout: post
title: "C++编译器什么时候为我们自动生成拷贝构造函数？"
date: 2012-11-25
comments: true
tags: Programming
---
<p>StackOverflow上有个一个有趣的问题，<a href="http://stackoverflow.com/questions/13374984/c-copy-by-value-to-function-params-produce-two-objects-in-vs2012/13551240#13551240">copy by value to function params produce two objects in vs2012</a></p>  <p>给出程序：</p> 
 
```cpp
#include <iostream>

using namespace std;

class A
{
public:
    A() { cout << "1" << endl; }
    ~A() { cout << "2" << endl; }
};

class B : public A
{
public:
    B() { cout << "3" << endl; }
    ~B() { cout << "4" << endl; }
};

void func(A a) {}

int main()
{
    B b;
    func(b);
    return 0;
}
```

<p>Visual Studio给出的输出是：132242。</p>

<p>前两个13和最后两个42.是对应的，分别是main函数中的局部变量b的构造和析构。问题出在中间的两个2，说明在调用函数func的时候创建了两个临时的A。</p>

<p>看到这个程序，第一眼的感觉应该是：因为func是参数是值传递，所以b传给func时，调用了A的拷贝构造函数，从b生成了一个A的临时变量，推出func时，这个临时变量析构，所以应该只有一个2.</p>

<p>&#160;</p>

<p>但是为什么会有两次呢？打开disassembly window，我们可以看到如下的汇编代码：</p>

```
B b;
00B317F8  lea         ecx,[b]  
00B317FB  call        B::B (0B31650h)  
00B31800  mov         dword ptr [ebp-4],0  
func(b);
00B31807  mov         al,byte ptr [ebp-12h]  
00B3180A  mov         byte ptr [ebp-13h],al  
00B3180D  mov         byte ptr [ebp-4],1  
00B31811  movzx       ecx,byte ptr [ebp-13h]  
00B31815  push        ecx  
00B31816  call        func (0B31730h)
```



<p>如果我们把原来的程序的中A的析构函数改成virtual，这样，输出的结果就是13242，中间只有1个2，说明只有一个临时的A被创建了出来，是我们期望的结果。汇编代码如下：</p>

```
B b;
 lea         ecx,[b]  
0033189B  call        B::B (03316A0h)  
003318A0  mov         dword ptr [ebp-4],0  
func(b);
003318A7  push        ecx  
003318A8  mov         ecx,esp  
003318AA  mov         dword ptr [ebp-1Ch],esp  
003318AD  lea         eax,[b]  
003318B0  push        eax  
003318B1  call        A::A (0331900h)  
003318B6  mov         dword ptr [ebp-20h],eax  
003318B9  call        func (03317D0h)
```

<p>回想Inside C++ object model中说到的只有在4中情况下编译器才会给我们生成缺省拷贝构造函数：</p>

<ol>
  <li>类包含的成员变量是object，并且这个object的类有拷贝构造函数。</li>

  <li>类继承自一个基类，这个基类有拷贝构造函数。</li>

  <li>类声明了1个或者多个虚函数。</li>

  <li>类继承自一个基类，这个基类有1个或者多个虚函数。</li>
</ol>

<p>所以在原来的代码中A不属于上面4类，所以编译器不会为它生成一个缺省拷贝构造函数。</p>

<p>我把我的理解写在了这个<a href="http://stackoverflow.com/a/13551240/304115">答案</a>里。</p>
---
layout: post
title: "没有初始化的指针会是个什么值呢？"
date: 2013-03-15
comments: true
tags: Programming
---
<p>在C++的构造函数中我们应该把所有成员变量都初始化，如果我们忘记了初始化一个成员指针，会发生什么呢？</p>  <p>假设有如下代码：</p>  

```cpp
class my
{
public:
    my(){};
    ~my(){delete[] r;}
private:
    float* r;
};

class my2
{
private:
    my m;
};

int main() {
    
    float* f;
    my m;
    my2* m2=  new my2();
    //delete[] f; 
    return 0;
}
```

<p>&#160;</p>

<p>Visual Studio 2012调试结果如下：</p>

<p><a href="http://images.cnitblog.com/blog/163228/201303/15163039-d46979d842144e87ad29c392b1593968.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://images.cnitblog.com/blog/163228/201303/15163039-cc8e1dfda0ce468bb5c76928b8b5eab1.png" width="244" height="60" /></a></p>

<p>Visual Studio 2008调试结果如下：</p>

<p><a href="http://images.cnitblog.com/blog/163228/201303/15163039-a1e66d95d84145f698b4fa9ec85f1653.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://images.cnitblog.com/blog/163228/201303/15163040-dac19fd40dd54735ac08a0e8b4a97007.png" width="244" height="56" /></a></p>



<p>&#160;</p>

<p>可以复习复习这个，<a href="/2012/07/06/what-is-oxcdcdcdcd/">oxcdcdcdcd是什么？</a></p>

<p>&#160;</p>

<p>如果把上面代码类my的构造函数和析构函数注释掉，那么类my和my2就变成了POD。POD就是Plain Old Data Structure，就是C++中没有用户自己定义的构造函数，析构函数和虚函数的类，并且每个成员也是POD。</p>

<p>Visual Studio2012和2008的调试结果都如下：</p>

<p><a href="http://images.cnitblog.com/blog/163228/201303/15163040-cb8d42aa8c0b4e759f212e352e64c13f.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://images.cnitblog.com/blog/163228/201303/15163041-06b4fdb3912c4a3ca501e22ff290c412.png" width="218" height="55" /></a></p>
---
layout: post
title: "C++怎么寻找重载函数"
date: 2012-12-10
comments: true
tags: Programming
---
<p><a href="http://accu.org/index.php/journals/268">ACCU :: Overload Resolution - Selecting the Function</a>这篇文章详细介绍了C++中寻找重载函数的方法。下面给个小例子吧，C++的重载有时候会违背你的直觉。</p>  <p>考虑如下代码：</p>  
  
```cpp
class OverLoadBase
{
public:
    int DoSomething() {return 0;};
};

class OverLoadSub : public OverLoadBase
{
public:
    int DoSomething(int x) {return 1;};
};

void TestOverLoadSub()
{
    OverLoadSub b;
    b.DoSomething(); //这样写会编译出错
}
```


<p>错误消息如下：</p>

```
In function 'void TestOverLoadSub()':
Line 17: error: no matching function for call to 'OverLoadSub::DoSomething()'
compilation terminated due to -Wfatal-errors.
```



<p>如果换成这样：</p>

```cpp
b.OverLoadBase::DoSomething(); 
```

<p>就没有问题了。或者把子类中的DoSomething（）去掉也没有问题。</p>

<p>为什么会这样呢？因为在上面那个例子中，两个DoSomething不是重载。上面的那篇文章里一开头就说“Declaring two or more items with the same name in a scope is called <em>overloading</em>”，只有在一个scope中才算重载。</p>
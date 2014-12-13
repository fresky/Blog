---
layout: post
title: "C++中子类的数组不能用父类指针来表示"
date: 2012-12-03
comments: true
tags: CPP
---
<p>假设我们有一个父类A，一个子类B，如果我们创建一个B的数组，我们能这样用吗？</p>

```cpp
A* barray = new B[10];
```
<p>写段代码在Visual Studio中来试试吧：）<span><br /></span></p>
<p>&nbsp;</p>

```cpp
#include <iostream>
#include <assert.h>

using namespace std;
class B;
class A
{
public:
    A();
    ~A();
    int aa;
};

class B:public A
{
public:
    B();
    ~B();
    int bb;
};

A::A()
{
    cout<<"A"<<" ";
    aa=1;
}

A::~A()
{
    cout<<"~A"<<" ";
    aa=-1;
}

B::B()
{
    cout<<"B"<<" ";
    bb=2;
}

B::~B()
{
    cout<<"~B"<<" ";

    bb=-2;
}

int main(int argc, char* argv[])
{
    cout<<"size of A: "<<sizeof(A)<<endl;
    cout<<"size of B: "<<sizeof(B)<<endl;
    A* aarray = new B[10];

    cout<<endl<<endl<<"aa value of A* is:"<<endl;
    for (int i = 0; i < 10; i++)
    {
        cout<<aarray[i].aa<<", ";
    }
    cout<<endl;

    B* baaray = (B*)aarray;
    cout<<endl<<"aa value of B* is:"<<endl;
    for (int i = 0; i < 10; i++)
    {
        cout<<baaray[i].aa<<", ";
    }
    cout<<endl<<endl;

    delete[] aarray;
    cout<<endl<<endl<<"After delete[]"<<endl;

    cout<<endl<<"aa value of A* is:"<<endl;
    for (int i = 0; i < 10; i++)
    {
        cout<<aarray[i].aa<<", ";
    }
    cout<<endl;
    baaray = (B*)aarray;
    cout<<endl<<"aa value of B* is:"<<endl;
    for (int i = 0; i < 10; i++)
    {
        cout<<baaray[i].aa<<", ";
    }
    return 0;
}
```
<p>&nbsp;</p>
<p>我们可以看到输出的结果是：</p>

```
size of A: 4
size of B: 8
A B A B A B A B A B A B A B A B A B A B

aa value of A* is:
1, 2, 1, 2, 1, 2, 1, 2, 1, 2,

aa value of B* is:
1, 1, 1, 1, 1, 1, 1, 1, 1, 1,

~A ~A ~A ~A ~A ~A ~A ~A ~A ~A

After delete[]

aa value of A* is:
4408224, -1, -1, -1, -1, -1, -1, -1, -1, -1,

aa value of B* is:
4408224, -1, -1, -1, -1, 1, 1, 1, 1, 1,
```

<p>说明尽管我们创建时用的是new B，但是因为声明的是A*，所以被认为是一个A的数组，按照A的大小来操作，所以可以看到成员变量aa的值是不对的。而且最后析构的时候只调用了A的析构函数，说明这个数组在delete的时候也是有问题的。</p>
<p>&nbsp;</p>
<p>如果我们把A的析构函数加上virtual，可以得到如下的结果：</p>

```
size of A: 8
size of B: 12
A B A B A B A B A B A B A B A B A B A B

aa value of A* is:
1, 668240, 2, 1, 668240, 2, 1, 668240, 2, 1,

aa value of B* is:
1, 1, 1, 1, 1, 1, 1, 1, 1, 1,

~B ~A ~B ~A ~B ~A ~B ~A ~B ~A ~B ~A ~B ~A ~B ~A ~B ~A ~B ~A

After delete[]

aa value of A* is:
-1, 668248, -2, -1, 668248, -2, -1, 668248, -2, -1,

aa value of B* is:
-1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
```

<p>说明只要我们加上virtual，在delete的时候编译器是知道怎么delete的。</p>
<p>&nbsp;</p>
<p>但是这并不是C++标准中制定的，不同的编译器会有不同的结果，比如如果把带virtual的代码放在g++中，会得到如下的结果：</p>

```
size of A: 8
size of B: 12
A B A B A B A B A B A B A B A B A B A B 

aa value of A* is:
1, 134540944, 2, 1, 134540944, 2, 1, 134540944, 2, 1, 

aa value of B* is:
1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 


Segmentation fault
```

<p>&nbsp;</p>
<p>所以，C++中子类的数组不能用父类指针来表示。</p>
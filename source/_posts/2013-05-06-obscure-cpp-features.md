---
layout: post
title: "C++的几个晦涩特性"
date: 2013-05-06
comments: true
tags: Programming
---
[Obscure C++ Features - Made by Evan](http://madebyevan.com/obscure-cpp-features/)列举了几个C++的晦涩特性，挺有意思的，下面简单列几个：

# []是啥意思

`ptr[3]`只是`(ptr + 3)`的简写，所以也就是`*(3 + ptr)`，所以它和`3[ptr]`是一样的。

# 运算符重载
运算符重载可以做很多奇怪的事情，比如可以通过运算符重载实现python style的`print`。如下：

```cpp
namespace __hidden__ {
    struct print {
        bool space;
        print() : space(false) {}
        ~print() { std::cout << std::endl; }

        template <typename T>
        print &operator , (const T &t) {
            if (space) std::cout << ' ';
            else space = true;
            std::cout << t;
            return *this;
        }
    };
}

#define print __hidden__::print(),

int main() {
    
     int a = 1, b = 2;

     print "this is a test";
     print "the sum of", a, "and", b, "is", a + b;

    return 0;
}
```

# 函数的try

可以在函数名称后面加上`try`，对应整个函数体。例如下面的代码。

```cpp
int f() { throw 0; }

// Here there is no way to catch the error thrown by f()
struct A {
  int a;
  A::A() : a(f()) {}
};

// The value thrown from f() can be caught if a try-catch block is used as
// the function body and the initializer list is moved after the try keyword
struct B {
  int b;
  B::B() try : b(f()) {
  } catch(int e) {
  }
  int B::C() try{
      throw 2;
      return 1;
  } catch(int e) {
      cout<<"exception "<<e<<endl;
  }
};
```
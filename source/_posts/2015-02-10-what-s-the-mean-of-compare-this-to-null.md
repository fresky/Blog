title: "判断this指针是不是null有什么意义呢"
date: 2015-02-10 18:32:02
categories:
tags: Programming
description: 不要在非静态函数中判断this指针是不是null，如果想支持空指针也能调用这个函数，就把这个函数定义为静态的。
---

今天看到一篇文章[Still Comparing "this" Pointer to Null?](http://www.viva64.com/en/b/0226/)，批评了判断this指针是不是null的做法。

文章中首先给出了如下的代码示例：
```c++
class CWindow {
    HWND handle;
    HWND GetSafeHandle() const
    {
         return this == 0 ? 0 : handle;
    }
};
```

这段代码看起来很奇怪，貌似目的只是为了允许一个空的`CWindow*`指针调用`GetSafeHandle`方法。

但是我们不应该这么做，首先C++的标准说明通过一个空指针调用非静态函数是未定义的行为。不过如下的代码可能能正常运行。

```c++
class Class {
public:
    void DontAccessMembers()
    {
        ::Sleep(0);
    }
};

int main()
{
    Class* object = 0;
    object->DontAccessMembers();
}
```

因为这个方法并没有使用任何成员变量，那么看起来它和静态函数差不多，所以调用可能是没问题的。

但是编译器可能会优化，比如最开头举得那个例子，可能编译器会认为this指针永远不会是null，这样就可能把判断this指针是不是空的判断优化掉。就像优化如下的if检查一样。

```c++
int divideBy = ...;
whatever = 3 / divideBy;
if( divideBy == 0 ) {
    // THIS IS IMPOSSIBLE
}
```

所以，你需要确保你的编译器不会做这样的优化，当然现在还没有任何编译器做这样的优化，但是未来不好说：）

另外，你如果看看如下代码的输出，估计会疯掉了：）

```c++
class FirstBase {
    int firstBaseData;
};

class SecondBase {
public:
    void Method()
    {
        if (this == 0) {
            printf("this == 0\r\n");
        }
        else {
            printf("this != 0 (value: %p)\r\n", this);
        }
    }
};

class Composed1 : public FirstBase, public SecondBase {
};

class Composed2 : public SecondBase, public FirstBase {
};


int main()
{
    Composed1* object1 = 0;
    object1->Method();

    Composed2* object2 = 0;
    object2->Method();
	
    SecondBase* object3 = reinterpret_cast<Composed1*>(0);
    object3->Method();

    SecondBase* object4 = reinterpret_cast<Composed1*>(1);
    object4->Method();
}
```

Visual Studio 2013的输出结果是：

```
this != 0 (value: 00000004)
this == 0
this == 0
this != 0 (value: 00000005)
Press any key to continue . . .
```

object1的this居然不是0，而是4，这是因为FirstBase有一个int的成员变量，在调用SecondBase的Method时，指针向后移了4！

object2的this就是0了，因为不需要向后移。

关于object3和object4的输出，就需要看看reinterpret_cast的汇编了。

```
SecondBase* object = reinterpret_cast<Composed1*>( rand() );
010C1000  call        dword ptr [__imp__rand (10C209Ch)] 
010C1006  test        eax,eax
010C1008  je          wmain+0Fh (10C100Fh) 
010C100A  add         eax,4 
object->Method();
010C100D  jne         wmain+20h (10C1020h) 
010C100F  push        offset string "this == 0" (10C20F4h) 
010C1014  call        dword ptr [__imp__printf (10C20A4h)] 
010C101A  add         esp,4 
```

可以看到那个je指令，如果是0，就会jump，这样就不会加4，如果不是0，就会加4。所以object3的输出是0，object4的输出是5。彻底崩溃了吧。

所以总结一下，不要在非静态函数中判断this指针是不是null，如果想支持空指针也能调用这个函数，就把这个函数定义为静态的。







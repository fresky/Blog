title: 在C#中我们能调用一个类的私有方法吗
date: 2015-04-03 01:03:09
categories:
tags: CSharp
description:
---
在C#中我们能调用一个类的私有方法吗？当然我们要排除掉用反射的方式：）直觉应该是不能。那我们来看一段代码吧。下面的代码修改自Eric Lippert的[Interface implementation in C# and VB](http://blog.coverity.com/2013/10/09/interface-implementation/)。

```c#
public interface MyInterface
{
	void Foo();
}
public class MyClass : MyInterface
{
	void MyInterface.Foo() // private by default
	{
		Console.WriteLine("MyClass Foo");
	}
}
class Program
{
	static void Main(string[] args)
	{
		MyClass myClass = new MyClass();
		//myClass.Foo();  // 编译错误
		MyInterface myClass2 = new MyClass();
		myClass2.Foo();
	}
}
```

只要我们用接口，就能访问`MyClass`的似有函数`Foo`了。这个稍微有点反直观，那么我们来看一个温和点的例子。

```c#
class MyClass2
{
	public static Action GetPrivateFoo()
	{
		return new Action(Foo);
	}
	private static void Foo()
	{
		Console.WriteLine("MyClass2 Foo!");
	}
}
class Program
{
	static void Main(string[] args)
	{
		var foo = MyClass2.GetPrivateFoo();
		foo();
	}
}	
```

这个有点好理解了，我们用一个公有方法把一个私有方法返回了出去，这样外面就能调用私有方法了。上面那个接口的例子其实和这个很类似。

那么`private`到底是什么意思呢？

**`private`表示这个方法不能被这个类之外通过查名字的方式看到，也就是说，`private`只是作为名字查找(name resolution)的输入，而不是一个成员能否被调用的限制**






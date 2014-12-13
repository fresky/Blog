---
layout: post
title: "定义Enum的开始和结束，这样就能循环Enum了"
date: 2012-07-17
comments: true
tags: CSharp
---

示例，这样还有个好处就是first=0成为一个不合法的enum，这样可以避免出现忘记初始化。但是要注意enum得顺序递增才能用loop。

```c#
	enum ProgrammingLanguage
    {
        Language_First = 0,
        CPP = 1,
        CSharp = 2,
        Java =3,
        Language_Last = 4,
    }
    class Program
    {
        static void Main(string[] args)
        {
           for (ProgrammingLanguage i = ProgrammingLanguage.Language_First + 1; i < ProgrammingLanguage.Language_Last; i++)
            {
                Console.WriteLine(i);
            }
        }
    }
```
---
layout: post
title: "C#的yield return是怎么被调用到的？"
date: 2013-02-27
comments: true
tags: CSharp
---
<p>假设有如下代码：</p>  

```csharp
		static IEnumerable<int> getInt()
        {
            for (int i = 0; i < 3; i++)
            {
                Console.WriteLine("get " + i);
                yield return i;
            }

        }
        
        static void Main(string[] args)
        {
            for (int i = 0; i < getInt().Count(); i++)
            {
                Console.WriteLine(getInt().ElementAt(i));
            }
        }
```

<p>输出结果是：</p>

```
get 0
get 1
get 2
get 0
0
get 0
get 1
get 2
get 0
get 1
1
get 0
get 1
get 2
get 0
get 1
get 2
2
get 0
get 1
get 2
```

<p>呵呵，说明了.Count()和ElementAt都是需要迭代的。</p>
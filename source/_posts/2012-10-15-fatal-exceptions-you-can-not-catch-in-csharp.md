---
layout: post
title: "C#中不能catch Fatal的exception"
date: 2012-10-15
comments: true
tags: Programming
---
<p><a href="http://vasters.com/clemensv/2012/09/06/Are+You+Catching+Falling+Knives.aspx">Clemens Vasters - Are you catching falling knives?</a>里给了一个判断C#的exception是不是fatal的代码，可以参考参考。</p>  

```csharp
public static bool IsFatal(this Exception exception)
{
    while (exception != null)
    {
        if (exception as OutOfMemoryException != null && exception as InsufficientMemoryException == null || exception as ThreadAbortException != null || 
            exception as AccessViolationException != null || exception as SEHException != null || exception as StackOverflowException != null)
        {
            return true;
        }
        else
        {
            if (exception as TypeInitializationException == null && exception as TargetInvocationException == null)
            {
                break;
            }
            exception = exception.InnerException;
        }
    }
    return false;
}
```
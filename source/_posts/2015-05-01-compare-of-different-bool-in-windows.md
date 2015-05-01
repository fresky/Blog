title: Windows上常见的集中布尔类型的比较
date: 2015-05-01 09:53:06
categories:
tags: Other
description:
---
[BOOL vs. VARIANT_BOOL vs. BOOLEAN vs. bool](blogs.msdn.com/b/oldnewthing/archive/2004/12/22/329884.aspx)解释了这几种布尔类型的来龙去脉。

写Windows1.0的时候C风头正盛，所以最老的`BOOL`是C风格的，它的定义如下：
```
typedef int BOOL;
```

接着OS/2 NT的开发人员引入了`BOOLEAN`，定义如下：
```
typedef BYTE BOOLEAN;
```

然后Visual Basic的人引入了`VARIANT_BOOL`，定义如下：**注意`VARIANT_TRUE`是-1！**
```
typedef short VARIANT_BOOL;
#define VARIANT_TRUE ((VARIANT_BOOL)-1)
#define VARIANT_FALSE ((VARIANT_BOOL)0)
```

`bool`是C++的数据类型，但是在Win32中基本看不见，因为Window2是C兼容的。
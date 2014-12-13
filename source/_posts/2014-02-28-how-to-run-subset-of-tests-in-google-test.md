---
layout: post
title: "如何在google test中指定只运行一部分测试"
date: 2014-02-28 08:47
comments: true
tags: [CPP, Testing]
keywords: google test,gtest
description: 如何在google test中用::testing::FLAGS_gtest_filter指定只运行一部分测试
---
[Google Test](https://code.google.com/p/googletest/)是一个很流行的C++测试框架，在使用google test的过程中，如果遇到了一个或者几个测试失败了，在调试的过程中，我们只想运行这些失败的测试，这个时候就可以用`::testing::FLAGS_gtest_filter`来指定。

在你的google test的`main`函数里写成如下代码：

```cpp
int _tmain(int argc, _TCHAR* argv[])
{
    ::testing::InitGoogleMock(&argc, argv);
	::testing::FLAGS_gtest_filter = "MyClassTest.testMyMethod*";
    return RUN_ALL_TESTS();
}
```
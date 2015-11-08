title: 如何检查机器是否因为装了Windows更新而需要重新启动
date: 2015-09-22 11:06:07
categories:
tags: [CPP, CSharp]
description:
---
今天的[The Old New Thing: How can I tell if Windows Update is waiting for the system to reboot?](http://blogs.msdn.com/b/oldnewthing/archive/2015/09/21/10642727.aspx)介绍了如何检查机器是否因为装了Windows更新而需要重新启动。下面是稍作修改的CPP和C#的代码。

```cpp
#include <atlstr.h>
#include <wuapi.h> 
#include <iostream>

int _tmain(int argc, _TCHAR* argv[])
{
    CoInitialize(NULL);
    {
        CComPtr<ISystemInformation> info;
        info.CoCreateInstance(CLSID_SystemInformation);

        VARIANT_BOOL rebootRequired;
        info->get_RebootRequired(&rebootRequired);

        std::cout << rebootRequired;
    }
    CoUninitialize();
	return 0;
}
```

```csharp
Type t = Type.GetTypeFromProgID("Microsoft.Update.SystemInfo");
object s = Activator.CreateInstance(t);
var needReboot = t.InvokeMember("RebootRequired", BindingFlags.GetProperty, null, s, null);
Console.WriteLine(needReboot);
```
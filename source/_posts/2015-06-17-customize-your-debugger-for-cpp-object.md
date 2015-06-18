title: 用Natvis定制C++对象在Visual Studio调试时如何显示
date: 2015-06-17 22:31:12
categories:
tags: [CPP, Debug]
description: 本文介绍如何使用Visual Studio的Natvis来定制如何在调试时显示C++对象。如果是C#对象，可以直接使用Debugger Display Attributes。
---
我在之前的博客[定制自己的Visual Studio的Debugger Visualizer](/2012/07/16/customize-your-own-debugger-visualizer-in-csharp/)和[如何把 Visutal studio中的“print-on-breakpoint”消息打印在程序的任何地方](/2012/07/13/print-message-to-anywhere-in-visual-studio-print-on-breakpoint/)中介绍过如何用[Debugger Display Attributes](https://msdn.microsoft.com/en-us/library/ms228992%28v=vs.110%29.aspx)来定制C#的对象在Visual Studio调试时如何显示，今天介绍一下如何通过[Natvis](https://msdn.microsoft.com/en-us/library/jj620914.aspx)对C++的对象做类似的事情。

首先看我们的示例的C++代码，一个类是用户，有一个成员变量是姓名，一个成员变量是懂得的技术。Main函数就是创建这个类的对象，并且赋值。

```c++
class User
{
public:
	User(string name);
	void AddSkill(string skill);

private:
	string m_Name;
	vector<string> m_Skills;
};

User::User(string name)
{
	m_Name = name;
}

void User::AddSkill(string skill)
{
	m_Skills.push_back(skill);
}

int _tmain(int argc, TCHAR* argv[])
{
	User u("Dawei");
	u.AddSkill("C++");
	u.AddSkill("C#");
    
	User u2("Tom");
	return 0;
}
```
在Main函数`return`的那一行打个断点，并且把`u`和`u2`加到Watch窗口，我们可以看到在Visual Studio中显示如下，这就是Visual Studio默认显示一个对象的方式。

{% limg natvis1.png %}

为了让这个对象显示的更加友好，我们可以写一个Natvis文件，告诉Visual Studio如何显示这个对象。我们的Natvis内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<AutoVisualizer xmlns="http://schemas.microsoft.com/vstudio/debugger/natvis/2010">
  <Type Name="User">
	<DisplayString Condition="m_Skills._Mylast - m_Skills._Myfirst == 0">{m_Name} has no skill!</DisplayString>
    <DisplayString>{m_Name} has {m_Skills._Mylast - m_Skills._Myfirst} skills!</DisplayString>
    <Expand>
		<ArrayItems>
		  <Size>m_Skills._Mylast - m_Skills._Myfirst</Size>
		  <ValuePointer>m_Skills._Myfirst</ValuePointer>
		</ArrayItems>
    </Expand>
  </Type>
</AutoVisualizer>
```

在这个Natvis中：
1. `Type`的`Name`表明这个Natvis针对那个对象，如果有namespace的话需要用`::`隔开，比如`Windows::UI::Xaml::Controls::TextBox`。也可以使用通配符*。  
1. `DisplayString`表明这个对象应该怎么显示。可以使用成员变量，但是**不能调用方法**，因为调用方法有可能会有副作用。  
1. `DisplayString`可以有条件判断，比如在这个例子中如果`m_Skills`的大小是0，就输出没有技能，如果不是0，就输出有几项技能。  
1. `Expand`用来表明如果展开这个对象，怎么显示。在这个例子中通过`ArrayItems`来把vector的每个元素都显示出来。  
1. 还有一些别的用法，比如可以用`UIVisualizer`来定制一个图像显示，就和我之前的那篇博文效果一样了。    

把这个Natvis存放在如下位置，Visual Studio就能找到了，每次修改之后不需要重启Visual Studio，只需要重新运行一次Debug就行了。
```
%VSINSTALLDIR%\Common7\Packages\Debugger\Visualizers (requires admin access)
%USERPROFILE%\My Documents\Visual Studio 2013\Visualizers\
```

如果发现没有起作用，可以在注册表中加入如下的键值，这样在Visual Studio的Output窗口就可以看到加载Natvis时的错误信息了。
```
[HKEY_CURRENT_USER\Software\Microsoft\VisualStudio\12.0_Config\Debugger]
"EnableNatvisDiagnostics"=dword:00000001 
```

加上这个Natvis之后，在Visual Studio中就可以看到如下的显示了。

{% limg natvis2.png %}
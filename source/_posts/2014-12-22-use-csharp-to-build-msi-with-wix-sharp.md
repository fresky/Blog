title: 用Wix#可以直接写C#代码来生成Wix的MSI安装文件
date: 2014-12-22 17:33:28
categories:
tags: CSharp
description: 如果不想写Wix的XML文件并且熟悉C#，那么现在有了一个新选择-Wix#。有了它，可以直接写C#代码来生成Wix的MSI安装文件。
---

如果不想写Wix的XML文件并且熟悉C#，那么现在有了一个新选择-[Wix#](https://wixsharp.codeplex.com/)。有了它，可以直接写C#代码来生成Wix的MSI安装文件。Wix#的作者Oleg Shilo 就是CS-Script的作者，我在之前的博客[用CS-Script把Notepad++变身支持智能提示和运行代码的C#集成开发环境](/2014/03/31/make-your-notepadd-plus-plus-as-csharp-ide-with-intellisense-and-code-execution/)介绍过。

使用方法非常简单，在C#的项目中添加nuget的Wixsharp依赖，会在你的项目中添加一个setup.cs包含示例代码，语法非常简洁明了。如下的示例代码在msi中加了2个文件，并且添加了一个customaction。

```c#
static public void Main(string[] args)
{
	Project project =
		new Project("MyProduct",
			new Dir(@"%ProgramFiles%\My Company\My Product",
				new File(@"Files\Bin\MyApp.exe"),
				new Dir(@"Docs\Manual",
					new File(@"Files\Docs\Manual.txt"))),
			new ManagedAction("MyAction"));

	project.UI = WUI.WixUI_InstallDir;
	project.GUID = new Guid("6f330b47-2577-43ad-9095-1861ba25889b");
	
	Compiler.BuildMsi(project);
}
```
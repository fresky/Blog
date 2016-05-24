title: 如何在Visual Studio中选择C++和C#的编译器版本
date: 2016-05-23 23:14:03
categories:
tags: [CSharp, CPP, Tool]
description: 本文介绍了Visual Studio的编译引擎MSBuild，以及如何使用Toolset来为MSBuild指定C#和C++的编译器。
---
# MSBuild简介
Visual Studio的编译引擎是[MSBuild](https://msdn.microsoft.com/en-us/library/dd393574.aspx)，它提供了一套项目文件（`.csproj`,`.vbproj`, `vcxproj`）的XML的Schema，用来指定如何处理和编译项目。

当然MSBuild不依赖于Visual Studio，完全可以在不安装Visual Studio的情况下使用MSBuild。比如可以从[Microsoft Build Tools 2015](https://www.microsoft.com/en-us/download/details.aspx?id=48159)下载MSBuild来编译C#。2016年3月31号微软也宣布了[Visual C++ Build Tools 2015](https://blogs.msdn.microsoft.com/vcblog/2016/03/31/announcing-the-official-release-of-the-visual-c-build-tools-2015/)，可以[下载](http://go.microsoft.com/fwlink/?LinkId=691126)来编译VC++的项目。

MSBuild也是一个MIT License的开源软件，可以在Github上看到它的[仓库](https://github.com/Microsoft/msbuild)。

[MSBuild Toolset (ToolsVersion)](https://msdn.microsoft.com/en-us/library/bb383796.aspx)是一个任务、目标和工具的集合，指定MSBuild的行为。通常一个MSBuild的Toolset包含`microsoft.common.tasks`文件，`microsoft.common.targets`文件和编译器比如`csc.exe`，`cl.exe`和`link.exe`。

在装Visual Studio时会按照对应版本的MSBuild，比如Visual Studio 2015对应的MSBuild就是v14。如果我们用Visual Studio创建C#和C++的项目，在`.csproj`和`.vcxproj`文件的第一行都指定了对应MSBuild的Toolset，如下所示。

```
<Project DefaultTargets="Build" ToolsVersion="14.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
```

在Visual Studio中编译项目时就会使用v14的MSBuild来编译。

# 用VS2015的MSBuild来编译VS2015创建的项目
我们也可以直接使用MSBuild来编译，我分别创建了一个空的C#和空的C++的Console应用，然后打开MSBuild Command Prompt for VS2015，用如下命令行来编译

```
msbuild CompilerDemo.sln /t:rebuild
```

可以看到如下的命令行输出：
```
CoreCompile:
  C:\Program Files (x86)\MSBuild\14.0\bin\csc.exe ...
  
ClCompile:
  C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\CL.exe ...

Link:
  C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\link.exe ...  
```

说明在编译项目时使用了v14的MSBuild下的`csc.exe`，`cl.exe`和`link.exe`。

# 用VS2013的MSBuild来编译VS2015创建的项目

如果我们打开Developer Command Prompt for VS2013，用VS2013的MSBuild编译项目。

```
msbuild CompilerDemo.sln /t:rebuild
```

得到如下的命令行输出：

```
CoreCompile:
  C:\Program Files (x86)\MSBuild\12.0\bin\Csc.exe ...
Done Building Project "D:\Documents\Visual Studio 2015\Projects\CompilerDemo\CSharpCompiler\CSharpCompiler.csproj" (Rebuild target(s)).

C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\V120\Microsoft.Cpp.Platform.targets(64,5): error MSB8020: The build tools for v140 (Platform Toolset = 'v140') cannot be found.
```

我们看到用v12的MSBuild编译C#项目时成功了，但是编译C++项目时失败了，说找不到Platform Toolset = 'v140'。我们回头打开`.vcxproj`文件，可以看到在这个文件里指定了`<PlatformToolset>v140</PlatformToolset>`。我们把它改成v120，再重新编译，这次发现成功了。

```
ClCompile:
  C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\CL.exe ...
Link:
  C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\link.exe ...
```

# 用VS2015的MSBuild来编译VS2015创建的项目（修改C++项目的PlatformToolset）
修改完`PlatformToolset`之后，我们再用VS2015的MSBuild编译一下，看看结果。

```
CoreCompile:
  C:\Program Files (x86)\MSBuild\14.0\bin\csc.exe ...
ClCompile:
  C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\CL.exe ...
Link:
  C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\link.exe ...
```
我们发现它使用了v14的`csc.exe`，但是使用了v12的`cl.exe`和`link`。

# 用VS2015的MSBuild来编译，但是指定VS2013的C++和C#的编译器

那有没有办法让VS2015的MSBuild也使用v12的C#的编译器（`csc.exe`）呢？我们可以参考一下[Overriding ToolsVersion Settings](https://msdn.microsoft.com/en-us/library/bb383985.aspx)。

可以使用MSBuild的命令行参数开关：`/ToolsVersion`，简写为`/tv`。打开MSBuild Command Prompt for VS2015，用如下命令行来编译。
```
msbuild CompilerDemo.sln /t:rebuild /tv:"12.0"
```
我们得到了如下的命令行输出：
```
CoreCompile:
  C:\Program Files (x86)\MSBuild\12.0\bin\Csc.exe ...
ClCompile:
  C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\CL.exe ...
Link:
  C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\link.exe ...
```

嗯，我们终于在VS2015的MSBuild命令下，成功的使用了VS2013的C++和C#的编译器。

# 用Visual Studio 2015来编译，但是指定VS2013的C++和C#的编译器
那么有没有办法来让Visual Studio 2015在编译时也用2013的编译器呢？我们没有办法来指定MSBuild的开关了。根据[Overriding ToolsVersion Settings](https://msdn.microsoft.com/en-us/library/bb383985.aspx)介绍的方法，需要做两件事。

## 修改项目文件的`ToolsVersion`属性

在本文的开头提到了在项目文件的第一行写明了对应MSBuild的Toolset。我们可以修改这个属性，把项目文件的第一行改成：
```
<Project DefaultTargets="Build" ToolsVersion="12.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
```

## 设置环境变量`MSBUILDLEGACYDEFAULTTOOLSVERSION`

```
set MSBUILDLEGACYDEFAULTTOOLSVERSION=true
```
这样在Visual Studio 2015中编译也会使用VS2013的编译器了。而且直接在MSBuild Command Prompt for VS2015中直接使用MSBuild，不需要制定`/tv`开关，也是使用VS2013的编译器了。

来验证一下，如果我们在代码中使用了C#6的新特性（VS2015支持）。
```csharp
object o = null;
Console.WriteLine(o?.ToString());
```

那么在Visual Studio 2015中编译会报错。

```
Build FAILED.

"D:\Documents\Visual Studio 2015\Projects\CompilerDemo\CompilerDemo.sln" (rebuild target) (1) ->
"D:\Documents\Visual Studio 2015\Projects\CompilerDemo\CSharpCompiler\CSharpCompiler.csproj" (Rebuild target) (2) ->
(CoreCompile target) -> 
  Program.cs(10,33): error CS1525: Invalid expression term '.' [D:\Documents\Visual Studio 2015\Projects\CompilerDemo\CSharpCompiler\CSharpCompiler.csproj]
  Program.cs(10,34): error CS1003: Syntax error, ':' expected [D:\Documents\Visual Studio 2015\Projects\CompilerDemo\CSharpCompiler\CSharpCompiler.csproj]
  Program.cs(10,44): error CS1002: ; expected [D:\Documents\Visual Studio 2015\Projects\CompilerDemo\CSharpCompiler\CSharpCompiler.csproj]
  Program.cs(10,44): error CS1525: Invalid expression term ')' [D:\Documents\Visual Studio 2015\Projects\CompilerDemo\CSharpCompiler\CSharpCompiler.csproj]

    0 Warning(s)
    4 Error(s)
```

# 在Visual Studio 2015中指定C#语言的版本
另外在Visual Studio中可以指定支持的C#语言的版本。右键项目-》属性-》编译-》高级-》语言版本。Visual Studio 2015的默认就是支持的最高版本C# 6.0，可以在这里选择不同的版本。

# 总结 TLDR

## 对于C++项目

**通过设置`PlatformToolset`来指定C++的编译器版本。**

## 对于C#项目

**使用相应版本的MSBuild。**

或者

**使用MSBuild的命令行开关`/ToolsVersion`（`/tv`）。**

或者

**1. 修改项目文件的`ToolsVersion`属性。**  
**2. 设置环境变量`MSBUILDLEGACYDEFAULTTOOLSVERSION`。**
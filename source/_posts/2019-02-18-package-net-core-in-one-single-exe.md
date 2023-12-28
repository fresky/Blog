title: 打包.NET Core的程序到一个单独的可执行文件
date: 2019-02-18 16:13:57
categories:
tags: Programming
description:
---
.NET Core的程序在发布时会是一个目录，里面放着exe和它的所有依赖。在一些情况下一个单独的EXE会更方便一些。[Warp](https://github.com/dgiagio/warp)是一个开源（MIT）的软件可以把`Node.js`，`.NET Core`和`Java`的程序打包成一个可执行文件，支持Linux,MacOS和Windows。使用也很方便，下面是Windows下打包.NET Core的命令（假设下载下来的Warp叫做warp-packer.exe）：

```
warp-packer.exe --arch windows-x64 --input_dir bin/Release/netcoreapp2.1/win10-x64/publish --exec myapp.exe --output myapp.exe
```

在这个之前需要先发布.NET Core的程序：

```
dotnet publish -c Release -r win10-x64
```

[dotnet-warp](https://github.com/Hubert-Rybak/dotnet-warp)是一个Global .NET Core的工具，简化了这个打包过程，用下面的命令安装dotnet-warp。

```
dotnet tool install --global dotnet-warp
```

然后直接在项目目录下运行下面的命令就够了。

```
dotnet-warp
```

这个工具现在有个小bug，如果你的项目目录包含空格，它会报如下的一个错误：

```
MSBUILD : error MSB1008: Only one project can be specified.
```
---
layout: post
title: "把C#当作脚本语言来用"
date: 2013-05-16
comments: true
tags: Programming
---
<p><a href="http://scottksmith.com/blog/2013/05/08/getting-started-with-scriptcs/">Getting started with ScriptCS</a>介绍<a href="http://scriptcs.net/">ScriptCS</a>，它能让你把把C#当作脚本语言来用。</p>  <p>这样你就能用脚本语言来写C#，比如把如下内容写在一个Sample.csx文件里。</p>  

```csharp
var x = 2;
var y = 3;
var z = x * y;
Console.WriteLine(z);
```

<p>然后在命令行写：</p>

```
scriptcs Sample.csx
```

<p>然后就能得到如下的输出：</p>

```
d:\>scriptcs Sample.csx
INFO : Starting to create execution components
INFO : Starting execution
6
INFO : Finished execution
```

<p><a href="https://github.com/scriptcs/scriptcs/wiki/Goals-and-Roadmap">这里</a>写了ScriptCS的Value和Roadmap。</p>

<h3>价值：</h3>

<p>1. 简单 - 不是要重复一个VS，不需要读文档</p>

<p>2. 灵活</p>

<p>3.快速开发的平台。</p>

<h3>用处：</h3>

<p>1. 快速原型开发</p>

<p>2. 构建简单应用</p>

<p>3. 写自动化脚本</p>

<p>4. 初始化程序，比如某些事的引导程序</p>

<p>5. 用脚本扩展一个应用</p>
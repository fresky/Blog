---
layout: post
title: "从博客园搬家到Github Pages"
date: 2013-09-24 17:01
comments: true
tags: [Github, CSharp]
---

前段时间开始在Github Pages上用Octopress写博客，可以参见[用octopress在Github pages上写博客](/2013/09/15/blog-in-github-pages-with-octopress/)。于是就想着把自己之前在博客园写的[博客](http://www.cnblogs.com/fresky/)也同步到Github上，就做了一个小工具[Blog2Github](https://github.com/fresky/Blog2Github)。

#使用方法
1. 下载[Blog2Github](https://raw.github.com/fresky/Blog2Github/master/Blog2Github.zip)，解压缩，运行Blog2Github.exe。
2. 如果使用的是博客园，就选择"cnblogs"。如果不是博客园，是其他的MetaWeblog，就选择其他，需要在下面的MetaWeblog URL里写明自己的博客的MetaWeblog地址.  
3. 输入博客的用户名和密码。  
4. 输入你想要迁移的博客文章数量。
5. 输入输出文件夹，通常应该是你的Octopress的source\\\_deploy目录，比如**d:\fresky.github.io\source\\_posts**)。
6. 点击"Generate"按钮，这样就会把你的博文以Octopress认识的格式存放在上面制定的输出目录。
7. 运行**rake deploy**命令来发布到Github Pages上。

参加下图：

![Blog2Github screenshot](https://raw.github.com/fresky/Blog2Github/master/screenshot.png)

#工作原理
这个小工具的工作原理很简单，就是通过MetaWeblog的API把你制定的博客文章都下载下来，然后在文章的开头插入Octopress需要的前缀如下，把博客标题和时间填上，这样Octopress就认识了。
```
---
layout: post
title: "从博客园搬家到Github Pages"
date: 2013-09-24 17:01
comments: true
tags: 
---
```

我用的是[MetaWeblogSharp](http://metaweblogsharp.codeplex.com/)提供的API，非常简单易用。

可以访问我的[博客](/archives/)看一下效果。

#已知问题
这个工具其实就是简单的把html写在了markdown文件里，大部分情况都能正确的处理，我遇到之前高亮过的代码因为可能html比较乱，markdown在处理的时候有些问题。

---
layout: post
title: "用octopress在Github pages上写博客"
date: 2013-09-15 21:16
comments: true
tags: [Github]
---

#安装Git环境

1. 下载[msysgit(git for windows)](https://code.google.com/p/msysgit/downloads/list)，并安装。
2. 可以选择安装[TortoiseGit](http://code.google.com/p/tortoisegit/)，这个在windows的资源管理器里装了很多git的右键菜单，对git命令行不熟悉的同学用起来很方便。

#安装Ruby环境


1. 从[RubyInstaller](http://www.rubyinstaller.org/downloads/)下载RubyInstaller，直接安装。
2. 从[RubyInstaller](http://www.rubyinstaller.org/downloads/)下载Devkit。  
注意下载合适的版本：  
    * Ruby 1.8.6 to 1.9.3: tdm-32-4.5.2
    * Ruby 2.0.0: mingw64-32-4.7.2
    * Ruby 2.0.0 x64 (64bits): mingw64-64-4.7.2
3. 下载的Devkit其实就是7zip的压缩包，双击时候选择解压位置，比如`c:\devkit`。
4. 在命令行运行如下命令安装devkit：  
```
	cd DevKit  
	ruby dk.rb init  
	ruby dk.rb install
```  
注意如果安装失败（不是两个[INFO]），可以在跑完`ruby dk.rb init`之后跑一下`ruby dk.rb review`来看看生成的`.yml`文件对不对，如果没有正确的找到ruby的安装目录，就手动加在下面。
5. 可以把gem的源更换到淘宝的镜像，这样比较快。  
```
gem sources -a http://ruby.taobao.org/  
gem sources -r https://rubygems.org/
``` 
然后可以`gem sources -l`看看结果。

#安装Octorpress

 1 . 首先把Octopress的代码拿到本地。
```
git clone git://github.com/imathis/octopress.git blog
```  
 2 . 修改Octopress目录下的Gemfile文件，将第一行的`http://rubygems.org/` 修改为`http://ruby.taobao.org/`  

 3 . 然后进入文件夹，安装Octopress。  
```
gem install bundler  
bundle install  
rake install
```
 4 . 解决中文问题。  
 
   * 在环境变量中设置下面的键值对：  
```
LANG=zh_CN.UTF-8
LC_ALL=zh_CN.UTF-8
```

   * 含有中文的文件需要保存为UTF-8无BOM格式编码。  
   * 在Ruby的安装路径找到文件convertible.rb，将27行修改为：  
    
```
self.content = File.read(File.join(base, name), :encoding => 'utf-8')
```  
 5 . 配置Octopress，修改`_config.yml`。重点修改如下几个：  
```
url: http://fresky.github.io
title: Flying in the free sky  
subtitle: Dawei's Blog
author: Dawei XU
simple_search: http://google.com/search  
description:
```
#写博客

博客必须存放在`source/_posts`目录下，并且满足[Jekyll](http://jekyllrb.com/)的命名规范：`YYYY-MM-DD-post-title.markdown`。
    
1 . 用`rake new_post["Title of the post"]`自动生成一个新的符合命名规范的博文。注意这里的标题不能有中文。

2 . 这个自动生成的文件的开头如下：
```
---
layout: post
title: "Tutorial: Create a blog with octopress and host it in github pages"  
date: 2013-04-22T21:24:21+02:00  
comments: true
tags: [ruby,octopress]
---
``` 
在这里可以修改title为中文。

3 . 用你喜欢的编辑器编辑Markdown文本。我用的是[MarkdownPad](http://markdownpad.com/)。    

4 . 编辑完之后可以运行`rake preview`来预览自己的博客（本地机器4000端口）。

#发布到Github Pages上

1 . 在Github上创建一个新的repository，名字叫做`your-username.github.io`。名字一定要符合这个规范。  

2 . 运行命令绑定到Github Pages上：  
```
rake setup_github_pages[git@github.com:fresky/fresky.github.com.git(reponame)]
```   
Github pages需要有2个分支，一个是`main`，一个是`source`。`main`上的内容用来显示。  

3 . 运行命令来发布：`rake deploy`

  
4 . 调用`git push origin source`来把你的博文的markdown也放到github上。

#在另一台机器上发布博文

Octopress的repository有2个分支，一个是`source`，相当于源代码，包含我们写的博文等文件，这些文件会被处理然后用来生成blog。另一个是`main`,包含博客本身。  
`main`分支存储在`_deploy`的目录下，这个目录以下划线开头，所以在`git push origin source`时会被忽略掉。当运行`rake deploy`时，会提交`master`。
所以我们要做的就是在新机器上重建一个Octopress的repository。

  1 . 克隆`source`到一个本地目录。
```
$ git clone -b source git@github.com:fresky/fresky.github.com.git blog
```

  2 . 克隆`master`到`_deploy`目录。

```
$ cd blog
$ git clone git@github.com:fresky/fresky.github.com.git _deploy 
```  

 3  . 运行rake来配置
```
$ gem install bundler
$ bundle install
$ rake setup_github_pages
```  
跟上面的步骤一样。
这样就在一个新的机器上设置好了Octopress的环境。

如果在多台机器上用，需要每次发布前都pull，发布完都push两个分支。

#参考
1. [Tutorial: Create a Blog With Octopress and Host It in Github Pages](http://miguelcamba.com/blog/2013/04/22/tutorial-create-a-blog-with-octopress-and-host-it-in-github-pages/)  
1. [Clone Your Octopress to Blog From Two Places](http://blog.zerosharp.com/clone-your-octopress-to-blog-from-two-places/)  
1. [Windows下搭建Octopress博客](http://www.cnblogs.com/oec2003/archive/2013/05/27/3100896.html)
  
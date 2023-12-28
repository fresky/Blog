title: 搭架私有Git服务器Gogs
date: 2015-11-07 00:50:35
categories:
tags: Tool
description:
---
我之前的博客[GitLab: 开源免费的git管理工具](/2013/06/08/gitlab-opensource-github-clone/)和[在Windows上安装私有GitHub的开源替代-GitLab](/2013/07/16/how-to-install-gitlab-in-windows-github-open-source-alternative/)介绍过如何自己搭建一个Git服务器来管理自己的私有仓库。今天再介绍一个Git服务器[Gogs (Go Git Service)](https://github.com/gogits/gogs)。

Gogs是一个用Go语言写的Git服务器，开源免费，MIT开源协议。因为Go是跨平台的，所有Gogs也支持所有平台。搭建Gogs非常简单，下面以Ubuntu为例介绍一下怎么安装Gogs。

1. [下载](https://github.com/gogits/gogs/wiki/Download)自己平台对应的二进制文件，是一个zip压缩包。在Ubuntu的终端运行`wget http://7d9nal.com2.z0.glb.qiniucdn.com/gogs_v0.6.15_linux_amd64.zip`。  
1. 解压，运行`unzip gogs_v0.6.15_linux_amd64.zip`。如果没有`unzip`命令，需要安装`sudo apt-get install unzip`。  
1. 到解压缩的目录下运行`./gogs web`，如果是远程ssh的，就运行`screen ./gogs web`。  
1. 打开浏览器输入`http://localhost:3000`就打开了安装界面，填一些配置选项就好了，支持多种数据库，如果只是几个人用，直接用sqlite最省事。  
1. 安装完后打开浏览器输入`http://localhost:3000`，就可以注册登录了。可以push上去一个repo试试，很简单。  
1. 如果更新Gogs，直接下载zip压缩包，删除老目录里的templates文件夹，别的直接解压覆盖就行了。  

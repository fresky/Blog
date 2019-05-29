title: 用opengrok搭建代码搜索引擎
date: 2019-05-29 23:15:42
categories:
tags: Tool
description:
---
[OpenGrok](https://oracle.github.io/opengrok/)是一个快速稳定的代码搜索引擎。它提供了一个[docker镜像](https://github.com/OpenGrok/docker)，可以通过如下的步骤很方便的搭建自己的代码搜索引擎。

1. 用`git clone`下载源代码，比如在`src`目录。  
2. 用如下的脚本自动pull最新的代码，可以设置成windows的一个schedue task。  
```
@echo off
cd src
set back=%cd%
for /d %%i in (%back%\*) do (
    echo git pull for repo: %%i
    cd "%%i"
    git pull
)
cd "%back%"
```
3. 运行如下的docker命令搭建代码搜索引擎。  
```
docker run -d -e REINDEX=<re-index minutes> -v <path/to/your/src>:/opengrok/src -p <host port>:8080 opengrok/docker:latest
```
其中REINDEX指定了多长时间重新创建索引，这个可以根据第二部pull代码的间隔来设置。如果不加这个参数的话默认是10分钟，如果设置成0就不会重新创建索引，这样适合在自己在本机临时起搜索引擎时使用。
title: 在Azure上搭架Shadowsocks代理服务器
date: 2015-12-19 18:00:59
categories:
tags: Resource
description:
---
简单总结一下如何在云服务的虚拟机中搭建Shadowsocks代理服务器。

1.注册一个云服务器提供商，比如美国Azure。  
2.创建一个Ubuntu的虚拟机。  
3.用SSH客户端（比如[putty](http://www.putty.org/)）连接虚拟机。  
3.安装[Shadowsocks](https://shadowsocks.org/en/index.html)。  
    1) 安装[PIP](https://pip.pypa.io)。 `sudo apt-get install python-pip`  
	2) 用PIP安装Shadowsocks。`sudo pip install shadowsocks`  
4.创建Shadowsocks的配置文件。`sudo vi vpn.json`，如下为示例内容。  
```json
{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
5.启动Shadowsocks服务。`ssserver -c /home/vpn/vpn.json -d start`。如果要停止服务`ssserver -c /home/vpn/vpn.json -d stop`  
6.记得在Azure的虚拟机配置中打开上面配置文件中指明的那个服务器端口8388。  
7.从Shadowsocks的[客户端](https://shadowsocks.org/en/download/clients.html)下载相应的客户端，所有平台都支持。比如Windows的在[这里](https://github.com/shadowsocks/shadowsocks-windows)，Android的在[这里](https://github.com/shadowsocks/shadowsocks-android)。这样就可以科学上网了。
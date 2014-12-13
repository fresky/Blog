---
layout: post
title: "把Silverlight的WCF service配置地址放到web.config中"
date: 2010-10-20
comments: true
tags: CSharp
---
如果在Silverlight中使用了WCF service，会生成一个ServiceReferences.ClientConfig配置文件，里面描述了WCF service的地址等信息。最后会放在生成的.xap中，可以用zip软件打开.xap，看到这个文件。<br /><br />如果开发时的WCF service和部署时的WCF service地址不一样，就需要每次部署时手动修改.xap包中的这个文件，比较麻烦而且容易出错。<br /><br />一个解决办法是把Service的地址放到web.config中，变成可配置的。然后在调用Siverlight的aspx或者html中通过initParms传给Silvergliht。下面是示例代码：<br /><br />aspx文件中：<br />

```xml
<param name="initParams" value="wcfServiceUrl=<%=ConfigurationManager.AppSettings["WCFServiceAddress"] %> " />
```

Silverlight文件初始化WCF Service是不用默认的构造函数，用这种方式：

```c#
BasicHttpBinding binding = new BasicHttpBinding(BasicHttpSecurityMode.None);
binding.MaxReceivedMessageSize = int.MaxValue;
binding.MaxBufferSize = int.MaxValue;
ServiceClient sc = new ServiceClient(binding, new EndpointAddress(new Uri(wcfServiceUrl)));
```
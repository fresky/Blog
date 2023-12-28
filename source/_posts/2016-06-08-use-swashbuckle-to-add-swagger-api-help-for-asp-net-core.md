title: 用Swashbuckle给ASP.NET Core的项目自动生成Swagger的API帮助文档
date: 2016-06-08 21:16:41
categories:
tags: [Tool, Programming]
description:
---
[Swagger](http://swagger.io/)是一个描述RESTful的Web API的规范和框架。如果使用ASP.NET的话，可以用[Swashbuckle](https://github.com/domaindrivendev/Swashbuckle)来自动生成Swagger。下面详细的介绍一下如何给ASP.NET Core的项目自动生成Swagger的API帮助文档。

# 创建ASP.NET Core的Web API Controller
在Visual Studio 2015中创建一个ASP.NET Core的项目，点击添加“New Item”，“Server-side”，“Web API Controller Class”。Visual Studio会帮我们自动创建一个如下的文件，实现了一个标准的RESTful的Web API。

```csharp
[Route("api/[controller]")]
public class ValuesController : Controller
{
    // GET: api/values
    [HttpGet]
    public IEnumerable<string> Get()
    {
        return new string[] { "value1", "value2" };
    }

    // GET api/values/5
    [HttpGet("{id}")]
    public string Get(int id)
    {
        return "value";
    }

    // POST api/values
    [HttpPost]
    public void Post([FromBody]string value)
    {
    }

    // PUT api/values/5
    [HttpPut("{id}")]
    public void Put(int id, [FromBody]string value)
    {
    }

    // DELETE api/values/5
    [HttpDelete("{id}")]
    public void Delete(int id)
    {
    }
}
```

# 添加Swashbuckle的Nuget包
打开`project.json`文件，添加Swashbuckle的依赖`Swashbuckle.SwaggerGen`和`Swashbuckle.SwaggerUi`。注意我们要使用6.0的版本，这是针对ASP.NET Core的。它的github地址[Ahoy](https://github.com/domaindrivendev/Ahoy)也和之前的版本不一样了。

```json
  "dependencies": {
    "Microsoft.AspNet.IISPlatformHandler": "1.0.0-rc1-final",
    "Microsoft.AspNet.Mvc": "6.0.0-rc1-final",
    "Microsoft.AspNet.Mvc.Core": "6.0.0-rc1-final",
    "Microsoft.AspNet.Server.Kestrel": "1.0.0-rc1-final",
    "Microsoft.AspNet.SignalR.Server": "3.0.0-rc1-final",
    "Microsoft.AspNet.StaticFiles": "1.0.0-rc1-final",
    "Swashbuckle.SwaggerGen": "6.0.0-rc1-final",
    "Swashbuckle.SwaggerUi": "6.0.0-rc1-final"
  },
```

# 在`Startup.cs`中配置Swashbuckle

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddSwaggerGen();
    // ...
}

public void Configure(IApplicationBuilder app)
{
    // ...
    app.UseSwaggerGen();
    app.UseSwaggerUi("help"); // API文档的地址，默认是 /swagger/ui
    // ...
}
```

# 运行项目，查看API文档，也能直接测试

万事俱备，运行项目，打开地址，就能看到如下的API文档了，还能直接在这里测试Web API。

{% limg swagger.png %}
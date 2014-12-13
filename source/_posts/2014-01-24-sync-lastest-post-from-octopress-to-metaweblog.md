---
layout: post
title: "把用octopress最新发布的博文同步到提供metaweblog API的博客（例如博客园）上"
date: 2014-01-24 08:43
comments: true
tags: [CSharp, Github]
keywords: octopress, metaweblog, csharp, password
description: 把用octopress最新发布的博文同步到提供metaweblog API的博客（例如博客园）上，在csharp的console application中读取密码。
---

最近逐渐开始用octopress在github pages上写博客，之前每次发完博客都用我自己写的那个windows live writer的[markdown插件](http://fresky.github.io/blog/2013/07/16/windows-live-writer-plugin-markdowninlivewriter/)发布到博客园上，但是觉得很麻烦，今天就动手写了个小程序SyncPost自动把最近的一篇博文同步到博客园上，使用metaweblog API。

程序很简单，使用前请先配置`SyncPost.exe.config`，就把主博客域名，本地的`octopress`的`_posts`目录，需要同步到的博客metaweblog地址，用户名和密码。如果关心信息安全，可以不写密码，这样程序在运行时会要求输入密码。

代码很少，可以到[Github](https://github.com/fresky/SyncPost)上看看。

下面简单介绍一下。

---
读密码，在控制台提示用户输入密码，然后用*遮盖，同时支持退格键，代码如下：
```c#
        private static string getPassword()
        {
            string password = System.Configuration.ConfigurationManager.AppSettings["Password"];
            if (!string.IsNullOrEmpty(password))
            {
                return password;
            }

            Console.Write("Password: ");
            ConsoleKeyInfo key;
            do
            {
                key = Console.ReadKey(true);
                if (key.Key != ConsoleKey.Backspace && key.Key != ConsoleKey.Enter)
                {
                    password += key.KeyChar;
                    Console.Write("*");
                }
                else
                {
                    if (key.Key == ConsoleKey.Backspace && password.Length > 0)
                    {
                        password = password.Substring(0, (password.Length - 1));
                        Console.Write("\b \b");
                    }
                }
            } while (key.Key != ConsoleKey.Enter);
            return password;
        }
```

---
从markdown中提取博文标题，根据文件名生成博文地址。

```c#
        private static void getLastestBlog(out string title, out string body)
        {
            title = "";
            string date = "";

            string fromBlog = System.Configuration.ConfigurationManager.AppSettings["FromBlog"];
            string postDir = System.Configuration.ConfigurationManager.AppSettings["PostDir"];
            DirectoryInfo di = new DirectoryInfo(postDir);
            FileInfo latestInfo = di.GetFiles("*.markdown").OrderByDescending(fi => fi.Name).First();

            using (StreamReader sr = new StreamReader(latestInfo.FullName))
            {
                while (!sr.EndOfStream)
                {
                    string line = sr.ReadLine();
                    if (line.StartsWith("title: "))
                    {
                        title = line.Substring(8, line.Length - 9);
                    }
                    else if (line.StartsWith("date: "))
                    {
                        date = line.Substring(6, 10);
                        break;
                    }
                }
            }

            string address = string.Format("{0}blog/{1}/{2}/", fromBlog, date.Replace('-', '/'),
                Path.GetFileNameWithoutExtension(latestInfo.Name).Substring(11));
            body =
                string.Format(
                    @"<p>博客搬到了<a href=""http://fresky.github.io/"">fresky.github.io - Dawei XU</a>，请各位看官挪步。最新的一篇是：<a href=""{0}"">{1}</a>。</p>",
                    address, title);
            Console.WriteLine("Original Link: {0}", address);
        }
```

---
发布。

```c#
        private static void postLastestBlog(string password, string title, string body)
        {
            string username = System.Configuration.ConfigurationManager.AppSettings["UserName"];
            string blogMetweblogUrl = System.Configuration.ConfigurationManager.AppSettings["ToBlog"];
            var blogcon = new BlogConnectionInfo(
                string.Empty,
                blogMetweblogUrl,
                string.Empty,
                username,
                password);

            var client = new Client(blogcon);
            client.NewPost(title, body, new List<string>(), true, null);
            Console.WriteLine("Done!");
        }
```
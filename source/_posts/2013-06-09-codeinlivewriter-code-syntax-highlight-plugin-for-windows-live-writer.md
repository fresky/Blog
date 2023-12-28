---
layout: post
title: "Windows Live Writer的一个代码语法高亮的插件——CodeInLiveWriter"
date: 2013-06-09
comments: true
tags: Programming
---
<p>&nbsp;</p>
<p>我平时经常用<a href="http://windows.microsoft.com/en-us/windows-live/essentials-other#essentials=overviewother">Windows Live Writer</a>来写博客，但是插入代码很麻烦，用了一些插件也都有些不满意的地方，我就自己写了一个，可以从<a href="http://msdn.microsoft.com/en-us/library/aa702851.aspx">这里</a>找到如何给Windows Live Writer写plugin。</p>
<p>我这个程序基于<a href="http://hilite.me/">hilite.me</a>提供的<a href="http://hilite.me/api">API</a>，hilite.me内部使用了<a href="http://pygments.org/">Pygments</a>，<a href="http://pygments.org/">Pygments</a>也被用于github的代码显示。</p>
<p>另外<a href="http://alexgorbatchev.com/SyntaxHighlighter/">SyntaxHighlighter</a>也是个被广泛应用的代码高亮的工具。</p>
<p>&nbsp;</p>
<p>使用方法：</p>
<p>下载<a href="https://github.com/fresky/CodeInLiveWriter/blob/master/CodeInLiveWriter.dll">CodeInLiveWriter.dll</a>，然后放在<em>[WindowsLiveWriterPath]\Plugins\</em>目录下就行了。源代码放在了<a href="https://github.com/fresky/CodeInLiveWriter" target="_blank">GitHub</a>上。</p>
<p>&nbsp;</p>
<p><strong>目前内建可选的语言有：</strong></p>
<ul>
<li>c#</li>
<li>c++</li>
<li>java</li>
<li>python</li>
<li>ruby</li>
<li>c</li>
<li>sql</li>
<li>html</li>
<li>xml</li>
<li>batch</li>
<li>bash</li>
</ul>
<p><strong>内建支持的格式：</strong></p>
<ul>
<li>vs</li>
<li>colorful</li>
<li>emacs</li>
<li>vim</li>
</ul>
<p>支持用户自定义语言和格式，分别存在 <strong>CodeInLiveWriterStyle.txt</strong> 和 <strong>CodeInLiveWriterLanguage.txt</strong>，然后和<strong>CodeInLiveWriter.dll</strong>放在同一个文件夹下。具体参见<a href="http://pygments.org/">Pygments</a>支持的<a href="http://pygments.org/docs/lexers/">语言</a> and <a href="http://pygments.org/docs/styles/">格式</a>。</p>
<p>&nbsp;</p>
<p>主要的代码如下：（就是用这个插件把代码插进来的：））

```csharp
        private string getHtmlStyleCode(string language, string code, bool showline, string style)
        {
            try
            {
                HttpWebRequest httpWebRequest = WebRequest.Create("http://hilite.me/api") as HttpWebRequest;
                httpWebRequest.Method = "POST";
                httpWebRequest.ContentType = "application/x-www-form-urlencoded";

                string parameters = string.Format("lexer={0}&code={1}&{2}&style={3}",
                                                  HttpUtility.UrlEncode(language),
                                                  HttpUtility.UrlEncode(code),
                                                  showline ? "&linenos=1" : "",
                                                  style);

                using (StreamWriter streamWriter = new StreamWriter(httpWebRequest.GetRequestStream()))
                {
                    streamWriter.Write(parameters);
                }
                string result;
                using (WebResponse response = httpWebRequest.GetResponse())
                {
                    using (StreamReader streamReader = new StreamReader(response.GetResponseStream()))
                    {
                        result = streamReader.ReadToEnd();
                    }
                }
                return result;
            }
            catch
            {
                return code;
            }
        }
```
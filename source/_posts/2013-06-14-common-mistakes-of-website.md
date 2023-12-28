---
layout: post
title: "网站设计中常见的几个错误"
date: 2013-06-14
comments: true
tags: Programming
---
<p>Scott Hanselman在他的博客<a href="http://www.hanselman.com/blog/StopDoingInternetWrong.aspx">Stop Doing Internet Wrong</a>上列举了几个网站设计中常见的错误。</p>  <ol>   <li>自动重定向到mobile页面。现在手机都这么先进，不如让用户选他是想看mobile页面还是正常页面。 </li>    <li>在浏览器中让用户打开app。 </li>    <li>巨大的广告。 </li>    <li>Label没有和checkbox关联，用户只能点那个小小的checkbox。其实如下的简单的html就能把label和checkbox关联起来。 （Scott在原文中有个typo，应该是小写的n）</li>    

```html
<p>Which fruit would you like for lunch?</p>
<form>
  <input type="radio" name="fruit" id="banana" />
  <label for="banana">Banana</label>
  <input type="radio" name="fruit" id="None" />
  <label for="none">None</label>
</form>
```

  <li>链接不好使。</li>

  <li>点国旗来选择语言。其实在http的request里面有accept-language，可以用这个。</li>

  <li>让用户输入邮编和省份。其实有很多<a href="http://stackoverflow.com/a/492978/304115">美国邮编API</a>可以做这件事。我也查了查，有很多web service提供中国的邮编查询，比如<a href="http://webservice.webxml.com.cn/WebServices/ChinaZipSearchWebService.asmx">这个</a>。</li>

  <li>通过设置高度和宽度来把一个巨大的图片放在页面上。（其实用户还是得下载它）你可以用<a href="http://pnggauntlet.com/">PNGGauntlet</a>或者<a href="http://advsys.net/ken/utils.htm">PNGOUT</a>来压缩你的png。</li>

  <li>应该通过重定向等技术让用户更容易访问你的网站。比如<a href="http://www.example.com">www.example.com</a>和example.com，或者/和/default.html.</li>
</ol>
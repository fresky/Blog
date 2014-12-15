---
layout: post
title: "允许OSQA上传任意类型的文件"
date: 2013-05-15
comments: true
tags: [Tool, Python]
---
<p>OSQA是一个很好的Q&amp;A的开源软件，我在之前的博客<a href="/2012/10/25/setup-osqa-in-windows/">用开源的OSQA在Windows上搭建Q&A网站</a>中写过怎么用Zoo在IIS中搭建。今天介绍一下怎么允许用户上传任意类型的文件。</p>  <p>默认的OSQA只允许上传图片，然后图片会直接显示在question和answer中。可以通过修改forum\views\writers.py这个文件，删掉下面2行代码。</p>  <p>   <div style="border-bottom: #cccccc 1px solid; border-left: #cccccc 1px solid; padding-bottom: 5px; background-color: #f5f5f5; padding-left: 5px; padding-right: 5px; border-top: #cccccc 1px solid; border-right: #cccccc 1px solid; padding-top: 5px" class="cnblogs_code">     <pre><span style="color: #0000ff">if</span> <span style="color: #0000ff">not</span> file_name_suffix <span style="color: #0000ff">in</span> (<span style="color: #800000">'</span><span style="color: #800000">.jpg</span><span style="color: #800000">'</span>, <span style="color: #800000">'</span><span style="color: #800000">.jpeg</span><span style="color: #800000">'</span>, <span style="color: #800000">'</span><span style="color: #800000">.gif</span><span style="color: #800000">'</span>, <span style="color: #800000">'</span><span style="color: #800000">.png</span><span style="color: #800000">'</span>, <span style="color: #800000">'</span><span style="color: #800000">.bmp</span><span style="color: #800000">'</span>, <span style="color: #800000">'</span><span style="color: #800000">.tiff</span><span style="color: #800000">'</span>, <span style="color: #800000">'</span><span style="color: #800000">.ico</span><span style="color: #800000">'</span><span style="color: #000000">):
            </span><span style="color: #0000ff">raise</span> FileTypeNotAllow()</pre>
  </div>
这样用户就可以通过点那个image按钮上传任意类型的文件了，这些文件也会和图片一样，放在forum\upfiles里面。这个时候是需要修改markdown，把![alt text][url]改成[text][url]，就可以了，上传的文件会作为一个链接存在，可以下载。

  </p>
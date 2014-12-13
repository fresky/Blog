---
layout: post
title: "gvim编辑xml更新"
date: 2010-10-09
comments: true
tags: Tool
---
上次写的那个gvim编辑xml的方式有点不太好，没有把xml的层级关系列出来，今天找了找，比较完美的解决了gvim编辑和查看xml的问题。<br /><br />查看xml，需要用到<a href="http://code.google.com/p/xmllint/">xmllint</a>，这个是windows版本的，需要放到path里面，其实就是在gvim里能找到就行。然后再gvim的配置中加上这个<br />let mapleader = ","<br />map &lt;silent&gt; &lt;leader&gt;px : %!xmllint % --format&lt;cr&gt;<br />这样在打开xml后，键盘输入,px，就能得到xml的pretty xml效果。<br /><br />对于编辑，需要<a href="http://www.vim.org/scripts/script.php?script_id=301">xmledit - A filetype plugin to help edit XML, HTML, and SGML documents</a> ，用这个覆盖掉原来gvim自带的xml.vim，在编辑xml文件时就能省很多事了，照抄文档里说。It allows you to jump to the beginning or end of the tag block your cursor is in. '%' will jump between '&lt;' and '&gt;' within the tag your cursor is in. When in insert mode and you finish a tag (pressing '&gt;') the tag will be completed. If you press '&gt;' twice it will complete the tag and place the cursor in the middle of the tags on it's own line. 
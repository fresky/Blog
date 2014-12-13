---
layout: post
title: "怎么用windbg查看managed dump里面的string的值"
date: 2013-05-12
comments: true
tags: Debug
---
<p><a href="http://stackoverflow.com/a/5352242/304115">stackoverflow</a>上这个回答很不错，用一个脚本把string的值打印到文本文件里，很方便。</p>  

```
$$ Dumps the managed strings to a file
$$ Platform x86
$$ Usage $$>a<"c:\temp\dumpstringtofolder.txt" 6544f9ac 5000 c:\temp\stringtest
$$ First argument is the string method table pointer
$$ Second argument is the Min size of the string that needs to be used filter
$$ the strings
$$ Third is the path of the file
.foreach ($string {!dumpheap -short -mt ${$arg1}  -min ${$arg2}})
{ 

  $$ MT        Field      Offset               Type  VT     Attr    Value Name
  $$ 65452978  40000ed        4         System.Int32  1 instance    71117 m_stringLength
  $$ 65451dc8  40000ee        8          System.Char  1 instance       3c m_firstChar
  $$ 6544f9ac  40000ef        8        System.String  0   shared   static Empty

  $$ start of string is stored in the 8th offset, which can be inferred from above
  $$ Size of the string which is stored in the 4th offset
  r@$t0=  poi(${$string}+4)*2
  .writemem ${$arg3}${$string}.txt ${$string}+8 ${$string}+8+@$t0
}
```

<p>把上面的代码村到一个文件里面，比如<code>c:\temp\dumpstringtofolder.txt。</code></p>

<p><code>然后在windbg里面敲如下的命令：</code></p>

```
$$>a<”c:\temp\dumpstringtofolder.txt” 6544f9ac 5000 c:\temp\stringtest
```


<font face="Courier New"></font>就能把string的值打印在c:\temp\stringtest+address.txt里。
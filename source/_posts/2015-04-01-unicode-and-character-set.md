title: Unicode和字符集小结
date: 2015-04-01 22:52:46
categories:
tags: Development
description:
---
今天又读了一遍Joel Spolsky的[The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets (No Excuses!)](http://www.joelonsoftware.com/articles/Unicode.html)，记一下笔记吧。

#ASCII字符集
ASCII字符集由95个可打印字符（0×20-0x7E）和33个控制字符（0×00-0×19，0x7F）组成。

#OEM字符集
一个字节是8位，如果用一个字节来表示ASCII字符集，那么多余的128个数字可以用来表示别的字符，所以128-255被称为OEM字符集，各种乱用，导致两台机器之间可能不能交流文档。

另外亚洲语言有上千个字符，根本不可能放进一个字节。通常通过双字节DBCS（Double Byte Character Set）来解决，有的字符用一个字节表示，有的用两个字节表示。

#ANSI标准
为了标准化，有了ANSI标准，128以上怎么使用取决于代码页（Code Page），从[Code Page](http://en.wikipedia.org/wiki/Code_page)可以查到各种语言的code page，比如中文就是936。

#Unicode

Unicode可以表示人类使用的所有字符，每个字符都由一个字符码（code point）来表示，在windows上可以用charmap这个小工具来看。

#编码

##UCS-2/UTF-16
如果我们用2个字节来表示Unicode，那么叫做UCS-2或者UTF-16（其实它是变长的，也可以是4个字节）。

为了区分大小端，引入了Unicode Byte Order Mark (BOM)，在Unicode字符串的开头加上FEFF。参见[Byte order mark](http://en.wikipedia.org/wiki/Byte_order_mark)

##UTF-8
因为英语文本中会引入很多00，所以就发明了UTF-8，0-127都还是用一个字节表示，所以英文在UTF-8和ASSCII，包括ANSI，OEM字符集的编码下看起来是一样的，而且还和以0作为字符串结尾兼容。

##UTF-7
UTF-7和UTF-8类似，但是保证最高位永远是0。

##UCS-4
UCS-4用4个字节表示，好处是任何字符的编码长度都是一样的。




传统的编码系统只能保存部分的code points，对于其他的code points，就会显示成问号。比如Windows-1252和ISO-8859-1（也就是Latin-1）。而UTF-7,8,16,32可以很好的存储任何的code points。

**如果给了一个字符串，但是没有给出他的编码方式，那是没有意义的。**

所以邮件就在邮件头中写明了编码方式，比如：
```html
Content-Type: text/plain; charset="UTF-8"。
```

对于网页，不能由网站在http头中写编码方式，因为一个网站有很多网页，可能每个网页的编码方式是不一样的，所以就需要写在html中。但是这就是鸡和蛋的关系了，需要知道编码方式从而来读取html，但是又需要读取html来获得编码方式。解决办法就是大家都遵守如下的约定，这样我们就能用ASCII码的方式来得到编码方式。

```html
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
```

但是假设你写网页的时候没有这么写，那么浏览器就只能猜了。如果没有猜对，用户可以通过浏览器菜单来选择编码方式。











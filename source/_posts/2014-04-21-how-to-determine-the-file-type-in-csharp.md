---
layout: post
title: "在C#中如何确定一个文件是不是文本文件，以及如何确定一个文件的类型"
date: 2014-04-21 18:43
comments: true
tags: CSharp
keywords: file type, csharp
description: 本文介绍如何在C#中通过检查文件中有没有\0来判断是不是文本文件，并且通过文件的前两个字节来判断文件类型。需要注意的是这两个方法都不是很准确。
---

今天需要从一个文本文件中读取信息，就想到如果用户选择的不是一个文本文件，遍历这个文件就是做无用功，于是就想能不能有一种方式可以判断文件是不是文本文件。在网上找到了一个方法，通过判断文件中有没有`\0`，如果有的话就一定不是文本文件。代码如下：
```csharp
public static bool IsTextFile(string path)
{
    // only check 32 char here, can increase to achieve higher accuracy
	var buf = new char[32];

	try
	{
		using (var reader = new StreamReader(path))
		{
			int readint = reader.ReadBlock(buf, 0, buf.Length);

			for (int i = 0; i < readint; i++)
			{
				if (buf[i] == '\0')
				{
					return false;
				}
			}
			return true;
		}
	}
	catch (Exception ex)
	{
		Console.WriteLine(ex);
	}
	return false;
}
```

针对一个更普遍的问题，如何根据文件内容来确定文件的格式，在网上找了另一个方法。读取文件的前两个字节，然后根据这个字节的值来做判断。代码如下：

```csharp
public static FileExtension CheckFileType(string path)
{
	string header = string.Empty;
	try
	{
		using (var fs = new FileStream(path, FileMode.Open, FileAccess.Read))
		{
			using (var r = new BinaryReader(fs))
			{
				byte buffer = r.ReadByte();
				header = buffer.ToString();
				buffer = r.ReadByte();
				header += buffer.ToString();
			}
		}
	}
	catch (Exception ex)
	{
		Console.WriteLine(ex.Message);
	}
	return (FileExtension) int.Parse(header);
}

public enum FileExtension
{
	JPG = 255216,
	GIF = 7173,
	BMP = 6677,
	PNG = 13780,
	COM = 7790,
	EXE = 7790,
	DLL = 7790,
	RAR = 8297,
	ZIP = 8075,
	XML = 6063,
	HTML = 6033,
	ASPX = 239187,
	CS = 117115,
	JS = 119105,
	TXT = 210187,
	SQL = 255254,
	BAT = 64101,
	BTSEED = 10056,
	RDP = 255254,
	PSD = 5666,
	PDF = 3780,
	CHM = 7384,
	LOG = 70105,
	REG = 8269,
	HLP = 6395,
	DOC = 208207,
	XLS = 208207,
	DOCX = 208207,
	XLSX = 208207,
}
```

但是这个方法有时候都不靠谱，比如我试了一个`.docx`的文件，返回值是`.zip`。不过我试了几个`.jpg`、`.exe`、`.dll`的都还行。另外我从网上搜了半天也没有搜到一个正儿八经的关于这两个字节和类型对应关系的出处，只是看到有很多网站用这个方法来判断用户上传的文件是不是图片。
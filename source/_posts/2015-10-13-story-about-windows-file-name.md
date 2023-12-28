title: 关于Windows文件名和路径名的那些事
date: 2015-10-13 22:42:02
categories:
tags: Programming
description: 本文介绍了Windows下关于文件名和路径名的一些规范，介绍了什么是8.3文件名，什么是长文件名和短文件名。并且介绍了如何绕过Windows关于路径260长度的限制。
---
[Naming Files, Paths, and Namespaces](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365247%28v=vs.85%29.aspx)介绍了Windows的文件、路径。要点如下：

# 文件名
## 文件命名规范
1. 用点`.`来分隔基本文件名和扩展名。  
1. 用反斜线`\`来分隔路径不同的组件。  
1. 盘符名中需要用反斜线`\`。比如`C:\path\file`中的`C:\`，UNC（Universal Naming Convention)下`\\server\share\path\file`中的`\\server\shar`。  
1. 不要假设区分大小写。有一些文件系统，比如POSIX兼容的文件系统，可能支持区分大小写。但是NTFS（支持POSIX）默认是不区分大小写的。  
1. 盘符不区分大小写。  
1. 用点`.`来表示当前目录。  
1. 用两个连续的点`..`来表示上层目录。  
1. 文件或者文件夹名字的末尾不能使点`.`或者空格。  
1. 文件名不能是系统预留的这些名字：`CON`, `PRN`, `AUX`, `NUL`, `COM1`, `COM2`, `COM3`, `COM4`, `COM5`, `COM6`, `COM7`, `COM8`, `COM9`, `LPT1`, `LPT2`, `LPT3`, `LPT4`, `LPT5`, `LPT6`, `LPT7`, `LPT8`和`LPT9`。也要避免直接在这些名字后面加一个扩展名。  
1. 除了以下的特殊字符外，可以在文件名中使用当前code page的任何字符，包括unicode字符。  

- 预留字符。包括`<`小于号， `>`大于号， `:`冒号， `"`双引号， `/`正斜线， `\`反斜线， `|`竖线， `?`问号， `*`星号。  
- ASCII中的0到31控制字符，详见[ASCII Code](http://www.ascii-code.com/)。  

## 短名字和长名字

首先说什么是**8.3文件名**。MS-DOS的FAT文件系统支持文件名最多8个字符加扩展名最多3个字符，加上中间的那个点，一共最多支持12个字符。

超过8.3文件名限制的文件名就叫做长名字，当创建一个长文件名的文件时，系统会生成一个相应的8.3文件名，叫做短文件名。

比如我们经常能看见在一些程序中文件名超过8个字符的，都会在最后包含一个`~`，这就是长文件名对应的短文件名。

Windows的API[GetShortPathName](https://msdn.microsoft.com/en-us/library/windows/desktop/aa364989%28v=vs.85%29.aspx)可以拿到一个长文件名对应的短文件名。[GetLongPathName](https://msdn.microsoft.com/en-us/library/windows/desktop/aa364980%28v=vs.85%29.aspx)可以拿到一个短文件名对应的长文件名。下面是一个示例的小程序。

```c++
int _tmain(int argc, _TCHAR* argv[])
{
	long length = 0;
	TCHAR* buffer = NULL;

	LPCWSTR path = TEXT("d:\\1111111111.txt");
	length = GetShortPathName(path, NULL, 0);

	if (length == 0)
	{
		_tprintf(FormatErrorMessage(GetLastError()));
	}

	buffer = new TCHAR[length];
	length = GetShortPathName(path, buffer, length);

	_tprintf(TEXT("long name = %s, short name = %s\r\n"), path, buffer);

	const int bufferSize = 100;
	TCHAR* longBuffer = new TCHAR[bufferSize];
	length = GetLongPathName(buffer, longBuffer, bufferSize);

	_tprintf(TEXT("long name = %s, short name = %s\r\n"), longBuffer, buffer);

	delete[] buffer;
	delete[] longBuffer;
	return 0;
}
```

它的输出如下，可以看到我有个文件的名字长度为10，超出了8.3文件名的限制，那么它对应的短文件名中就会包含一个`~`。

```
long name = d:\1111111111.txt, short name = d:\111111~1.TXT
long name = d:\1111111111.txt, short name = d:\111111~1.TXT
```

其中的`FormatErrorMessage`是拿到`GetLastError`的[错误码](https://msdn.microsoft.com/en-us/library/windows/desktop/ms681381%28v=vs.85%29.aspx)的对应消息。加入我的D盘下没有对应的1111111111.txt这个文件，那么会输出错误信息`The system cannot find the file specified.`。代码如下：

```c++
CString FormatErrorMessage(DWORD ErrorCode)
{
	TCHAR   *pMsgBuf = NULL;
	DWORD   nMsgLen = FormatMessage(FORMAT_MESSAGE_ALLOCATE_BUFFER |
		FORMAT_MESSAGE_FROM_SYSTEM | FORMAT_MESSAGE_IGNORE_INSERTS,
		NULL, ErrorCode, MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT),
		(LPTSTR)(&pMsgBuf), 0, NULL);
	if (!nMsgLen)
		return _T("FormatMessage fail");
	CString sMsg(pMsgBuf, nMsgLen);
	LocalFree(pMsgBuf);
	return sMsg;
}
```

# 路径

文件的路径由多个组件组成，用反斜线`\`分隔。路径的前缀指明了路径的命名空间。如果有个组件是文件名，那么它一定是最后一个组件。

## 相对路径和绝对路径

以下面这些开头的路径是绝对路径：

1. UNC名字，两个反斜杠开头`\\`。  
1. 盘符冒号加反斜杠开头，例如`C:\`。  
1. 一个反斜杠开头，例如`\file.txt`。

注意：如果一个路径是盘符加冒号开头，但是没有反斜杠，那么**它是相对路径**。所以`C:1.txt`表示的意思是C盘下的当前目录下面的1.txt文件。因为Windows每个盘符都记录了一个当前路径，那么可以用这种方式来快速在不同盘符的当前路径下切换。同样的`C:..\1.txt`表示C盘的当前目录的上层目录下的1.txt文件。

## 最长路径名限制

Windows下的最长路径长度是**260**字符。但是Windows下的很多API支持扩展长度的路径名，支持最长为**32767**的路径。这些路径要以`\\?\`作为前缀，如果是UNC格式，那么就以`\\?\UNC\`为前缀。

这个长路径由多个组件（文件夹或者文件）组成，每个组件的长度不能超过[GetVolumeInformation](https://msdn.microsoft.com/en-us/library/windows/desktop/aa364993(v=vs.85).aspx)返回的`lpMaximumComponentLength`。（这个值通常是255）

如下是一个支持长路径的例子，首先在D盘创建了一个名字长为244的文件夹，然后在这个文件夹下面又创建了一个名字长为244的文件，如果不加`\\?\`作为前缀，那么文件会创建失败，错误码为3，ERROR_PATH_NOT_FOUND，The system cannot find the path specified。如果加上`\\?\`作为前缀，就能创建成功了。

```c++
int _tmain(int argc, _TCHAR* argv[])
{
	LPCWSTR folder = L"d:\\111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111";
	bool ret = CreateDirectory(folder, NULL);
	if (!ret)
	{
		printf("Could not create folder %ws, error %d\n", folder, GetLastError());
	}

	LPCWSTR name = L"\\\\?\\d:\\111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111\\111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111.txt";
	//LPCWSTR name = L"d:\\111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111\\111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111.txt";

	HANDLE handle = (HANDLE)CreateFile(name, GENERIC_WRITE, FILE_SHARE_WRITE, NULL, CREATE_ALWAYS, NULL, NULL);
	if (handle == INVALID_HANDLE_VALUE)
		printf("Could not create file %ws, error %d\n", name, GetLastError());
	CloseHandle(handle);

	return 0;
}
```
















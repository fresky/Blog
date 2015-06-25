title: 聊聊LAA（LARGE ADDRESS AWARE）
date: 2015-06-25 00:46:30
categories:
tags: [CSharp, Debug, Tool]
description: 本文介绍了什么是LAA（LARGE ADDRESS AWARE），怎么设置LAA，怎么检查LAA。并且介绍了Windows PE格式。
---
#什么是LAA（LARGE ADDRESS AWARE）
如果我们的应用程序是32位的，那么即使在64位的Windows上运行，默认也最多只能用2G的物理内存。升级程序成64位不是一件特别容易的事情，在升级之前，可以用先LAA（LARGE ADDRESS AWARE）技术来让应用程序可以使用4G内存。

[LAA（LARGE ADDRESS AWARE）](https://msdn.microsoft.com/en-us/library/wz223b1z.aspx)是应用程序的一个选项，它告诉操作系统，这个应用程序可以处理大于2G的内存。

#Windows的内存和地址空间限制
[Memory Limits for Windows and Windows Server Releases](https://msdn.microsoft.com/en-us/library/aa366778.aspx)介绍了Windows各个系统上不同内存类型可用的内存限制，下面表格列举了不同位的应用程序在不同位的操作系统上的内存限制。

<table>
	<tbody>
		<tr>
			<th>内存类型</th>
			<th>32位Windows操作系统</th>
			<th>64位Windows操作系统</th>
		</tr>
		<tr>
			<td>
				<p>32位程序用户态虚拟地址空间</p>
			</td>
			<td>
				<p>2G</p>
				<p>最多3G（指定IMAGE_FILE_LARGE_ADDRESS_AWARE和4GT）</p>
			</td>
			<td>
				<p>2G (没指定IMAGE_FILE_LARGE_ADDRESS_AWARE） (默认)</p>
				<p>4G (指定IMAGE_FILE_LARGE_ADDRESS_AWARE）</p>
			</td>
		</tr>
		<tr>
			<td>
				<p>64位程序用户态虚拟地址空间</p>
			</td>
			<td>
				<p>不能运行</p>
			</td>
			<td>
				<p>指定IMAGE_FILE_LARGE_ADDRESS_AWARE(默认):</p>
				<p>x64:&nbsp;&nbsp;8 TB</p>
				<p>Intel Itanium-based systems:&nbsp;&nbsp;7 TB</p>
				<p>Windows&nbsp;8.1 and Windows Server&nbsp;2012&nbsp;R2:&nbsp;&nbsp;128 TB</p>
				<p>2G (没指定IMAGE_FILE_LARGE_ADDRESS_AWARE）</p>
			</td>
		</tr>
		<tr>
			<td>
				<p>内核态虚拟地址空间</p>
			</td>
			<td>
				<p>2G</p>
				<p>1G到2G （指定4GT）</p>
			</td>
			<td>
				<p>8 TB</p>
				<p>Windows&nbsp;8.1 and Windows Server&nbsp;2012&nbsp;R2:&nbsp;&nbsp;128 TB</p>
			</td>
		</tr>
	</tbody>
</table>

##什么是4GT（4 Gigabyte Tuning）
[4GT](https://technet.microsoft.com/en-us/library/cc786709%28WS.10%29.aspx)也叫**/3GB**开关，是一种允许增加用户态应用程序可用内存大小的技术。32位Windows只能使用最多4G内存，一般是2G内存给内核态，2G内存给用户态。4GT技术通过降低内核态使用的内存（最多降到1G）来增大用户态可以使用的内存。

4GT只适用于32位Windows。
##什么是PAE（Physical Address Extension）
[PAE（Physical Address Extension）](https://msdn.microsoft.com/en-us/library/aa366796.aspx)是另外一种处理器的功能，它可以允许特定版本的32位Windows使用超过4G的物理内存。

PAE只适用于32位Windows。

#如何设置LAA（LARGE ADDRESS AWARE）
##给C++程序设置LAA（LARGE ADDRESS AWARE）
在程序的链接选项中制定**Enalbe Large Addresses**，如下图所示。
{% limg LAA4CPP.png %}
##给C#程序设置LAA（LARGE ADDRESS AWARE）
用`editbin`命令。
```
editbin /LARGEADDRESSAWARE <your exe>
```
#设置LAA对程序做了什么（Windows PE文件格式简介）
上面那些设置到底对应用程序做了什么呢？这就需要了解一些[Windows Portable Executable](https://en.wikipedia.org/wiki/Portable_Executable)文件格式了。PE文件包含下面几个组成部分，详细信息可以参见[Microsoft Portable Executable and Common Object File Format Specification](http://download.microsoft.com/download/e/b/a/eba1050f-a31d-436b-9281-92cdfeae4b45/pecoff.doc)。也可以直接看`winnt.h`头文件，里面有PE格式的详细定义。

![PE figure from osdev](http://wiki.osdev.org/images/d/dd/PEFigure1.jpg)

##DOS Stub
PE文件的第一个部分是DOS Stub，它包含1个DOS头和一个DOS可执行代码。

DOS头的开始是`e_magic`，值一定是`0x5A4D`（是ASCII码的Mark Zbikowski首字母缩写，Mark Zbikowski是DOS系统的架构师）。DOS头偏移0x3C的位置是`e_lfanew (File address of new exe header)`，它表明了PE头的位置。可以参加`_IMAGE_DOS_HEADER`的定义。

DOS的可执行代码就只是简单的输出`"This program cannot be run in DOS mode."`。

##PE头
通过上面的`e_lfanew`可以找到PE头，它一开始就是一个固定值`0x4550`。接着是文件头，定义是[IMAGE_FILE_HEADER](https://msdn.microsoft.com/en-us/library/windows/desktop/ms680313%28v=vs.85%29.aspx)。
```c++
typedef struct _IMAGE_FILE_HEADER {
    WORD    Machine;
    WORD    NumberOfSections;
    DWORD   TimeDateStamp;
    DWORD   PointerToSymbolTable;
    DWORD   NumberOfSymbols;
    WORD    SizeOfOptionalHeader;
    WORD    Characteristics;
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```

PE头的开始表明程序可以运行的机器类型。它的取值范围是：
```
IMAGE_FILE_MACHINE_I386, 0x014c	
IMAGE_FILE_MACHINE_IA64, 0x0200
IMAGE_FILE_MACHINE_AMD64, 0x8664
```

PE头偏移`0x12`是`Characteristics`，它表明了这个应用程序的一些属性。其中一个属性就是`IMAGE_FILE_LARGE_ADDRESS_AWARE(0x0020)`，它表明这个程序可以处理大于2G的内存。

设置应用程序的LAA（LARGE ADDRESS AWARE）就是在应用程序的PE头里面设置这个属性。

#如何检查应用程序有没有设置LAA（LARGE ADDRESS AWARE）
知道了设置LAA（LARGE ADDRESS AWARE）对程序做了什么，那么怎么检查应用程序有没有设置LAA（LARGE ADDRESS AWARE）就非常简单了，我们只需要检查文件的PE头，看看`Characteristics`里有没有设置`IMAGE_FILE_LARGE_ADDRESS_AWARE(0x0020)`这个属性。

##用Dumpbin
首先可以使用[DUMPBIN ](https://msdn.microsoft.com/en-us/library/c1h23y6c.aspx)命令。
```
dumpbin /headers <your exe>
```
会列出文件的PE头信息，如下所示：
```
FILE HEADER VALUES
             14C machine (x86)
               6 number of sections
        557E68AE time date stamp Mon Jun 15 13:54:54 2015
               0 file pointer to symbol table
               0 number of symbols
              E0 size of optional header
             123 characteristics
                   Relocations stripped
                   Executable
                   Application can handle large (>2GB) addresses
                   32 bit word machine
```
				   
##用PE Insider
也可以使用一些专门的查看PE头的工具来看。比如可以用[PE Insider](http://cerbero.io/peinsider/)，它是一个免费的看PE头的工具，非常方便，如下所示。
{% limg peinsider.png %}

可以点击Description来看具体的`Characteristics`，如下所示：
{% limg peinsider_characteristics.png %}

##用代码
当然我们也可以用代码来打开应用程序检查。下面是C#的示例代码。

```c#
static bool IsLargeAware(string file)
{
	const int DOS_MZ_HEADER = 0x5A4D;
	const int PE_HEADER_OFFSET = 0x3C;
	const int NT_HEADER = 0x4550;
	const int CHARACTERISTICS_OFFSET = 0x12;
	const int IMAGE_FILE_LARGE_ADDRESS_AWARE = 0x20;

	using (var fileStream = File.OpenRead(file))
	{
		var binaryReader = new BinaryReader(fileStream);
		if (binaryReader.ReadInt16() != DOS_MZ_HEADER)
			return false;
		
		binaryReader.BaseStream.Position = PE_HEADER_OFFSET;
		var peLocation = binaryReader.ReadInt32(); 

		binaryReader.BaseStream.Position = peLocation;
		if (binaryReader.ReadInt32() != NT_HEADER) 
			return false;

		binaryReader.BaseStream.Position += CHARACTERISTICS_OFFSET;
		return (binaryReader.ReadInt16() & IMAGE_FILE_LARGE_ADDRESS_AWARE) == IMAGE_FILE_LARGE_ADDRESS_AWARE;
	}
}
```
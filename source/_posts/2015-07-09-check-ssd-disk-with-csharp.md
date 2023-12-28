title: 如何用C#检查硬盘是否是固态硬盘SSD
date: 2015-07-09 22:00:48
categories:
tags: Programming
description: 本文介绍了如何用C#来查看硬盘是不是“no seek penalty”，或者nominal media rotation rate是1，从而判断硬盘是不是固态硬盘SSD。
---
在我的上一篇文章[用C#来查看电脑硬件和系统信息](/2015/07/08/know-your-computer-from-csharp/)中介绍了`Environment`和`ManagementClass`两个类。那我们能不能通过这两个类得到硬盘是不是固态硬盘SSD呢？答案是否定的。

那我们怎么才能判断一个硬盘是不是固态硬盘SSD呢？我们知道Windows的磁盘碎片整理对SSD是默认关闭的，那么Windows是怎么判断硬盘是不是固态硬盘呢？[Windows 7 Disk Defragmenter User Interface Overview](http://blogs.technet.com/b/filecab/archive/2009/11/25/windows-7-disk-defragmenter-user-interface-overview.aspx)介绍了磁盘碎片整理的算法，它通过下面两种方式来判断硬盘是不是固态硬盘SSD。

1. Volumes on disks whose driver reports “no seek penalty”.就是说如果硬盘没有查找开销，那么就认为是SSD。具体定义如下：

> Disk port driver needs to send a valid response to `IOCTL_STORAGE_QUERY_PROPERTY` call. Disk Defragmenter issues `IOCTL_STORAGE_QUERY_PROPERTY` request with `QueryType = PropertyStandardQuery` and `PropertyId = StorageDeviceSeekPenaltyProperty`.

> A valid response involves populating a `DEVICE_SEEK_PENALTY_DESCRIPTOR` structure with accurate version and size data. If the `IncursSeekPenalty` member is set to `false`, the disk is considered a SSD.

2. Volumes on ATA/SATA disks that report a nominal media rotation rate of 1.就是说如果硬盘的转速是1，就认为是固态硬盘SSD。具体定义如下：

> If the port driver does not return valid data for `StorageDeviceSeekPenaltyProperty`, Disk Defragmenter looks at the result of directly querying the device through the ATA IDENTIFY DEVICE command. Defragmenter issues `IOCTL_ATA_PASS_THROUGH` request and checks `IDENTIFY_DEVICE_DATA` structure. If the `NomimalMediaRotationRate` is set to 1, this disk is considered a SSD.

> The latest SSDs will respond to the command by setting word 217 (which is used for reporting the nominal media rotation rate to 1).  The word 217 was introduced in 2007 in the ATA8-ACS specification.

我们也可以用类似的方法来判断硬盘是不是SSD，下面的代码修改自[
Tell whether SSD or not in C#](https://emoacht.wordpress.com/2012/11/06/csharp-ssd/)。主要函数是`isSSDBySeekPenalty`和`isSSDByNominalMediaRotationRate`。

```
// For CreateFile to get handle to drive
private const uint GENERIC_READ = 0x80000000;
private const uint GENERIC_WRITE = 0x40000000;
private const uint FILE_SHARE_READ = 0x00000001;
private const uint FILE_SHARE_WRITE = 0x00000002;
private const uint OPEN_EXISTING = 3;
private const uint FILE_ATTRIBUTE_NORMAL = 0x00000080;

// CreateFile to get handle to drive
[DllImport("kernel32.dll", SetLastError = true)]
private static extern SafeFileHandle CreateFileW(
	[MarshalAs(UnmanagedType.LPWStr)]
	string lpFileName,
	uint dwDesiredAccess,
	uint dwShareMode,
	IntPtr lpSecurityAttributes,
	uint dwCreationDisposition,
	uint dwFlagsAndAttributes,
	IntPtr hTemplateFile);

// For control codes
private const uint FILE_DEVICE_MASS_STORAGE = 0x0000002d;
private const uint IOCTL_STORAGE_BASE = FILE_DEVICE_MASS_STORAGE;
private const uint FILE_DEVICE_CONTROLLER = 0x00000004;
private const uint IOCTL_SCSI_BASE = FILE_DEVICE_CONTROLLER;
private const uint METHOD_BUFFERED = 0;
private const uint FILE_ANY_ACCESS = 0;
private const uint FILE_READ_ACCESS = 0x00000001;
private const uint FILE_WRITE_ACCESS = 0x00000002;

private static uint CTL_CODE(uint DeviceType, uint Function,
							 uint Method, uint Access)
{
	return ((DeviceType << 16) | (Access << 14) |
			(Function << 2) | Method);
}

// For DeviceIoControl to check no seek penalty
private const uint StorageDeviceSeekPenaltyProperty = 7;
private const uint PropertyStandardQuery = 0;

[StructLayout(LayoutKind.Sequential)]
private struct STORAGE_PROPERTY_QUERY
{
	public uint PropertyId;
	public uint QueryType;
	[MarshalAs(UnmanagedType.ByValArray, SizeConst = 1)]
	public byte[] AdditionalParameters;
}

[StructLayout(LayoutKind.Sequential)]
private struct DEVICE_SEEK_PENALTY_DESCRIPTOR
{
	public uint Version;
	public uint Size;
	[MarshalAs(UnmanagedType.U1)]
	public bool IncursSeekPenalty;
}

// DeviceIoControl to check no seek penalty
[DllImport("kernel32.dll", EntryPoint = "DeviceIoControl",
		   SetLastError = true)]
[return: MarshalAs(UnmanagedType.Bool)]
private static extern bool DeviceIoControl(
	SafeFileHandle hDevice,
	uint dwIoControlCode,
	ref STORAGE_PROPERTY_QUERY lpInBuffer,
	uint nInBufferSize,
	ref DEVICE_SEEK_PENALTY_DESCRIPTOR lpOutBuffer,
	uint nOutBufferSize,
	out uint lpBytesReturned,
	IntPtr lpOverlapped);

// For DeviceIoControl to check nominal media rotation rate
private const uint ATA_FLAGS_DATA_IN = 0x02;

[StructLayout(LayoutKind.Sequential)]
private struct ATA_PASS_THROUGH_EX
{
	public ushort Length;
	public ushort AtaFlags;
	public byte PathId;
	public byte TargetId;
	public byte Lun;
	public byte ReservedAsUchar;
	public uint DataTransferLength;
	public uint TimeOutValue;
	public uint ReservedAsUlong;
	public IntPtr DataBufferOffset;
	[MarshalAs(UnmanagedType.ByValArray, SizeConst = 8)]
	public byte[] PreviousTaskFile;
	[MarshalAs(UnmanagedType.ByValArray, SizeConst = 8)]
	public byte[] CurrentTaskFile;
}

[StructLayout(LayoutKind.Sequential)]
private struct ATAIdentifyDeviceQuery
{
	public ATA_PASS_THROUGH_EX header;
	[MarshalAs(UnmanagedType.ByValArray, SizeConst = 256)]
	public ushort[] data;
}

// DeviceIoControl to check nominal media rotation rate
[DllImport("kernel32.dll", EntryPoint = "DeviceIoControl",
		   SetLastError = true)]
[return: MarshalAs(UnmanagedType.Bool)]
private static extern bool DeviceIoControl(
	SafeFileHandle hDevice,
	uint dwIoControlCode,
	ref ATAIdentifyDeviceQuery lpInBuffer,
	uint nInBufferSize,
	ref ATAIdentifyDeviceQuery lpOutBuffer,
	uint nOutBufferSize,
	out uint lpBytesReturned,
	IntPtr lpOverlapped);

// For error message
private const uint FORMAT_MESSAGE_FROM_SYSTEM = 0x00001000;

[DllImport("kernel32.dll", SetLastError = true)]
static extern uint FormatMessage(
	uint dwFlags,
	IntPtr lpSource,
	uint dwMessageId,
	uint dwLanguageId,
	StringBuilder lpBuffer,
	uint nSize,
	IntPtr Arguments);

// Method for error message
private static string GetErrorMessage(int code)
{
	StringBuilder message = new StringBuilder(255);

	FormatMessage(
	  FORMAT_MESSAGE_FROM_SYSTEM,
	  IntPtr.Zero,
	  (uint)code,
	  0,
	  message,
	  (uint)message.Capacity,
	  IntPtr.Zero);

	return message.ToString();
}

// Method for no seek penalty
private static bool isSSDBySeekPenalty(string logicalDrive)
{
	string sDrive = "\\\\.\\" + logicalDrive + ":";

	SafeFileHandle hDrive = CreateFileW(
		sDrive,
		0, // No access to drive
		FILE_SHARE_READ | FILE_SHARE_WRITE,
		IntPtr.Zero,
		OPEN_EXISTING,
		FILE_ATTRIBUTE_NORMAL,
		IntPtr.Zero);

	if (hDrive == null || hDrive.IsInvalid)
	{
		string message = GetErrorMessage(Marshal.GetLastWin32Error());
		Console.WriteLine("CreateFile failed. " + message);
	}

	uint IOCTL_STORAGE_QUERY_PROPERTY = CTL_CODE(
		IOCTL_STORAGE_BASE, 0x500,
		METHOD_BUFFERED, FILE_ANY_ACCESS); // From winioctl.h

	STORAGE_PROPERTY_QUERY query_seek_penalty = new STORAGE_PROPERTY_QUERY();
	query_seek_penalty.PropertyId = StorageDeviceSeekPenaltyProperty;
	query_seek_penalty.QueryType = PropertyStandardQuery;

	DEVICE_SEEK_PENALTY_DESCRIPTOR query_seek_penalty_desc = new DEVICE_SEEK_PENALTY_DESCRIPTOR();

	uint returned_query_seek_penalty_size;

	bool query_seek_penalty_result = DeviceIoControl(
		hDrive,
		IOCTL_STORAGE_QUERY_PROPERTY,
		ref query_seek_penalty,
		(uint)Marshal.SizeOf(query_seek_penalty),
		ref query_seek_penalty_desc,
		(uint)Marshal.SizeOf(query_seek_penalty_desc),
		out returned_query_seek_penalty_size,
		IntPtr.Zero);

	hDrive.Close();

	if (query_seek_penalty_result == false)
	{
		string message = GetErrorMessage(Marshal.GetLastWin32Error());
		Console.WriteLine("DeviceIoControl failed. " + message);
	}
	else
	{
		if (query_seek_penalty_desc.IncursSeekPenalty == false)
		{
			return true;
		}
	}
	return false;
}

// Method for nominal media rotation rate
// (Administrative privilege is required)
private static bool isSSDByNominalMediaRotationRate(string logicalDrive)
{
	string sDrive = "\\\\.\\" + logicalDrive + ":";
	SafeFileHandle hDrive = CreateFileW(
		sDrive,
		GENERIC_READ | GENERIC_WRITE, // Administrative privilege is required
		FILE_SHARE_READ | FILE_SHARE_WRITE,
		IntPtr.Zero,
		OPEN_EXISTING,
		FILE_ATTRIBUTE_NORMAL,
		IntPtr.Zero);

	if (hDrive == null || hDrive.IsInvalid)
	{
		string message = GetErrorMessage(Marshal.GetLastWin32Error());
		Console.WriteLine("CreateFile failed. " + message);
	}

	uint IOCTL_ATA_PASS_THROUGH = CTL_CODE(
		IOCTL_SCSI_BASE, 0x040b, METHOD_BUFFERED,
		FILE_READ_ACCESS | FILE_WRITE_ACCESS); // From ntddscsi.h

	ATAIdentifyDeviceQuery id_query = new ATAIdentifyDeviceQuery();
	id_query.data = new ushort[256];

	id_query.header.Length = (ushort)Marshal.SizeOf(id_query.header);
	id_query.header.AtaFlags = (ushort)ATA_FLAGS_DATA_IN;
	id_query.header.DataTransferLength =
		(uint)(id_query.data.Length * 2); // Size of "data" in bytes
	id_query.header.TimeOutValue = 3; // Sec
	id_query.header.DataBufferOffset = (IntPtr)Marshal.OffsetOf(
		typeof(ATAIdentifyDeviceQuery), "data");
	id_query.header.PreviousTaskFile = new byte[8];
	id_query.header.CurrentTaskFile = new byte[8];
	id_query.header.CurrentTaskFile[6] = 0xec; // ATA IDENTIFY DEVICE

	uint retval_size;

	bool result = DeviceIoControl(
		hDrive,
		IOCTL_ATA_PASS_THROUGH,
		ref id_query,
		(uint)Marshal.SizeOf(id_query),
		ref id_query,
		(uint)Marshal.SizeOf(id_query),
		out retval_size,
		IntPtr.Zero);

	hDrive.Close();

	if (result == false)
	{
		string message = GetErrorMessage(Marshal.GetLastWin32Error());
		Console.WriteLine("DeviceIoControl failed. " + message);
	}
	else
	{
		// Word index of nominal media rotation rate
		// (1 means non-rotate device)
		const int kNominalMediaRotRateWordIndex = 217;

		if (id_query.data[kNominalMediaRotRateWordIndex] == 1)
		{
			return true;
		}
	}
	return false;
}
```


title: 如何用C#来检查硬盘是不是SSD固态硬盘
date: 2018-03-20 08:34:07
categories:
tags: Programming
description:
---
[Tell whether SSD or not in C#](https://emoacht.wordpress.com/2012/11/06/csharp-ssd/)介绍了怎么用C#来判断一个磁盘是不是SSD固态硬盘。文中使用的方法来源于微软一篇介绍Windows7磁盘碎片整理的文章[Windows 7 Disk Defragmenter User Interface Overview](https://blogs.technet.microsoft.com/filecab/2009/11/25/windows-7-disk-defragmenter-user-interface-overview/)，在这篇文章中指出Windows7在做磁盘碎片整理的时候会把SSD的磁盘排除在外，它用了3种检测方法（如果前面一种方法失败了会尝试下面一种方法）：  
* 没有seek penalty  
* nominal media rotation rate是1  
* random read disc score高于一个经验阈值（使用[Windows System Assessment Tool (WinSAT)](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc742157(v=ws.11))方法。  

主要就是用Windows的[`CreateFile`](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363858(v=vs.85).aspx)API来打开磁盘设备，然后调用[`DeviceIoControl`](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363216(v=vs.85).aspx)API来获取这些信息。

详见下面的代码，除了判断是不是SSD，也顺带用[`Management`](https://msdn.microsoft.com/en-us/library/system.management.managementclass(v=vs.110).aspx)获取机器的硬件信息，包括：机器名，MAC地址，序列号，Model，Manufacturer，本地用户等信息。

```csharp

static void Main(string[] args)
{
    GetPhysicalDriveNumberFromLogicalDriveLetter();

    //CheckSSDForPhysicalDrive();

    //string version = "1.0";
    //string computername = "UNKNOWN";
    //string servicetag = "UNKNOWN";
    //string wifimac = "UNKNOWN";
    //string model = "UNKNOWN";
    //string manufacturer = "UNKNOWN";

    ////  Get Manufacturer
    //manufacturer = GetManufacturer();
    ////  Get Model
    //model = GetModel();
    ////  Get computer name
    //computername = GetComputerName();
    ////  Get the wifi MAC address
    //wifimac = GetWifiMAC();
    ////  Get the service tag
    //servicetag = GetServiceTag();
    ////  Write header to console
    //Console.WriteLine("*************************************");
    //Console.WriteLine("ASSET TAGGING VERSION {0}", version);
    //Console.WriteLine("*************************************");
    ////  Write info to console
    //Console.WriteLine("Make: {0}", manufacturer);
    //Console.WriteLine("Model: {0}", model);
    //Console.WriteLine("Computer Name: {0}", computername);
    //Console.WriteLine("Wireless MAC: {0}", wifimac);
    //Console.WriteLine("Service Tag: {0}", servicetag);
    ////  Wait for user to press a key
    //Console.WriteLine();
    //Console.WriteLine("Press any key to exit");
    //Console.ReadKey();
}

private static void CheckSSDForPhysicalDrive()
{
    Console.WriteLine("Input physical drive number:");
    string sDrive = "\\\\.\\PhysicalDrive" + Console.ReadLine();

    Console.WriteLine(
        "Select method (0: No seek penalty, 1: Nominal media rotation rate):");
    switch (int.Parse(Console.ReadLine()))
    {
        case 0:
            HasNoSeekPenalty(sDrive);
            break;

        case 1:
            HasNominalMediaRotationRate(sDrive);
            break;
    }

    Console.ReadLine();
}

private static string GetComputerName()
{
    return Environment.MachineName;
}

private static string GetWifiMAC()
{
    string MAC = "UNKNOWN";

    foreach (NetworkInterface nic in NetworkInterface.GetAllNetworkInterfaces())
    {
        if (nic.NetworkInterfaceType == NetworkInterfaceType.Wireless80211)
        {
            MAC = nic.GetPhysicalAddress().ToString();
            //  Insert hyphens
            MAC = String.Join("-", Regex.Matches(MAC, @"\w{2}").Cast<Match>());
        }
    }
    return MAC;
}

private static string GetServiceTag()
{
    string servicetag = "UNKNOWN";

    ManagementClass wmi = new ManagementClass("Win32_Bios");
    foreach (ManagementObject bios in wmi.GetInstances())
    {
        servicetag = bios.Properties["Serialnumber"].Value.ToString().Trim();
    }
    return servicetag;
}

private static string GetModel()
{
    string model = "UNKNOWN";

    ManagementClass wmi = new ManagementClass("Win32_ComputerSystem");
    foreach (ManagementObject computer in wmi.GetInstances())
    {
        model = computer.Properties["Model"].Value.ToString().Trim();
    }
    return model;
}

private static string GetManufacturer()
{
    string manufacturer = "UNKNOWN";

    ManagementClass wmi = new ManagementClass("Win32_ComputerSystem");
    foreach (ManagementObject computer in wmi.GetInstances())
    {
        manufacturer = computer.Properties["Manufacturer"].Value.ToString().Trim();
    }
    return manufacturer;
}

private static string GetLocalUser()
{
    string localUser = "UNKNOWN";

    SelectQuery query = new SelectQuery("Select * from Win32_UserAccount Where LocalAccount=True");
    ManagementObjectSearcher searcher = new ManagementObjectSearcher(query);
    foreach (ManagementObject envVar in searcher.Get())
    {
        localUser += envVar["Name"] + ",";
    }
    return localUser;
}

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
private const uint IOCTL_VOLUME_BASE = 0x00000056;

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


// Method for no seek penalty
private static void HasNoSeekPenalty(string sDrive)
{
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

    STORAGE_PROPERTY_QUERY query_seek_penalty =
        new STORAGE_PROPERTY_QUERY();
    query_seek_penalty.PropertyId = StorageDeviceSeekPenaltyProperty;
    query_seek_penalty.QueryType = PropertyStandardQuery;

    DEVICE_SEEK_PENALTY_DESCRIPTOR query_seek_penalty_desc =
        new DEVICE_SEEK_PENALTY_DESCRIPTOR();

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
            Console.WriteLine("This drive has NO SEEK penalty.");
        }
        else
        {
            Console.WriteLine("This drive has SEEK penalty.");
        }
    }
}

// Method for nominal media rotation rate
// (Administrative privilege is required)
private static void HasNominalMediaRotationRate(string sDrive)
{
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
            Console.WriteLine("This drive is NON-ROTATE device.");
        }
        else
        {
            Console.WriteLine("This drive is ROTATE device.");
        }
    }
}

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


// For DeviceIoControl to get disk extents
[StructLayout(LayoutKind.Sequential)]
private struct DISK_EXTENT
{
    public uint DiskNumber;
    public long StartingOffset;
    public long ExtentLength;
}

[StructLayout(LayoutKind.Sequential)]
private struct VOLUME_DISK_EXTENTS
{
    public uint NumberOfDiskExtents;
    [MarshalAs(UnmanagedType.ByValArray)]
    public DISK_EXTENT[] Extents;
}

// DeviceIoControl to get disk extents
[DllImport("kernel32.dll", EntryPoint = "DeviceIoControl",
            SetLastError = true)]
[return: MarshalAs(UnmanagedType.Bool)]
private static extern bool DeviceIoControl(
    SafeFileHandle hDevice,
    uint dwIoControlCode,
    IntPtr lpInBuffer,
    uint nInBufferSize,
    ref VOLUME_DISK_EXTENTS lpOutBuffer,
    uint nOutBufferSize,
    out uint lpBytesReturned,
    IntPtr lpOverlapped);


private static void GetPhysicalDriveNumberFromLogicalDriveLetter()
{
    Console.WriteLine("Input logical drive letter:");
    char cDrive = Console.ReadLine()[0];
    if (char.IsLetter(cDrive))
    {
        GetDiskExtents(cDrive);
    }

    Console.ReadLine();
}

// Method for disk extents
private static void GetDiskExtents(char cDrive)
{
    DriveInfo di = new DriveInfo(cDrive.ToString());
    if (di.DriveType != DriveType.Fixed)
    {
        Console.WriteLine("This drive is not fixed drive.");
    }

    string sDrive = "\\\\.\\" + cDrive.ToString() + ":";

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

    uint IOCTL_VOLUME_GET_VOLUME_DISK_EXTENTS = CTL_CODE(
        IOCTL_VOLUME_BASE, 0,
        METHOD_BUFFERED, FILE_ANY_ACCESS); // From winioctl.h

    VOLUME_DISK_EXTENTS query_disk_extents =
        new VOLUME_DISK_EXTENTS();

    uint returned_query_disk_extents_size;

    bool query_disk_extents_result = DeviceIoControl(
        hDrive,
        IOCTL_VOLUME_GET_VOLUME_DISK_EXTENTS,
        IntPtr.Zero,
        0,
        ref query_disk_extents,
        (uint)Marshal.SizeOf(query_disk_extents),
        out returned_query_disk_extents_size,
        IntPtr.Zero);

    hDrive.Close();

    if (query_disk_extents_result == false ||
        query_disk_extents.Extents.Length != 1)
    {
        string message = GetErrorMessage(Marshal.GetLastWin32Error());
        Console.WriteLine("DeviceIoControl failed. " + message);
    }
    else
    {
        Console.WriteLine("The physical drive number is: " +
                            query_disk_extents.Extents[0].DiskNumber);
    }
}

```
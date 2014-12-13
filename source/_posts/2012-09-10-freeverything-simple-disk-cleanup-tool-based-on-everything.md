---
layout: post
title: "FreeEverything-基于everything的一个简易磁盘清理工具"
date: 2012-09-10
comments: true
tags: [CSharp, Tool]
---
<p>用Visual Studiode attach to process调试时，无论你有没有设置symbol path，Visual Studio都会把下载的symbol乱放，特别是会放到solution下面，导致文件夹很乱，所以写了一个小工具来删除这些symbol文件夹。同时也能删除resharper和mstest的临时文件。</p>
<p><a href="http://www.voidtools.com/">Everything</a>是一个非常快的磁盘搜索，正好基于他的<a href="http://support.voidtools.com/everything/SDK">SDK</a>来做这个小工具。</p>
<p>界面如下：</p>

{% limg freeeverything.png %}

<p>搜索因为是基于Everything的，所以很快。可以增加新的过滤，用正则表达式搜索，这些过滤存放在启动目录下的GarbageCan.xml里面。还能对搜索到的结果计算大小（如果文件很多会比较慢）。</p>
<p>源代码放在<a href="https://github.com/fresky/FreeEverything">Github</a>上，可执行文件在<a href="https://github.com/fresky/FreeEverything/blob/master/FreeEverything.zip">这里</a>下载。</p>
<p>1. 使用<a href="http://www.galasoft.ch/mvvm/">MVVMLight</a>这个mvvm的框架，这个框架很容易上手。</p>
<p>2. 使用XmlSerializer存储filter配置文件。</p>
<p>3. 检查everything是否启动。</p>

```c#
		public static void StartEverything()
        {
            Regex regex = new Regex(@"Everything([-.0-9])");
            bool found = false;
            foreach (var process in Process.GetProcesses())
            {
                if (regex.Match(process.ProcessName).Success)
                {
                    found = true;
                    break;
                }

            }
            if (!found)
            {
                ProcessStartInfo processStartInfo = new ProcessStartInfo(@"ThirdParty\Everything.exe");
                processStartInfo.CreateNoWindow = true;
                processStartInfo.WindowStyle = ProcessWindowStyle.Hidden;
                Process.Start(processStartInfo);
                m_EverythingLaunch = true;
            }
        }
```

<p>4. 调用Everything SDK. Everything SDK上面就有C#的示例project，非常简单。</p>
<p>5. 删除文件。需要处理readonly的问题。</p>

```c#
		private static void deletePath(string path)
        {
            FileSystemInfo fsi;
            if (File.Exists(path))
            {
                fsi = new FileInfo(path);
            }
            else if (Directory.Exists(path))
            {
                fsi = new DirectoryInfo(path);

            }
            else
            {
                return;
            }
            deleteFileSystemInfo(fsi);
        }
        private static void deleteFileSystemInfo(FileSystemInfo fsi)
        {

            fsi.Attributes = FileAttributes.Normal;
            var di = fsi as DirectoryInfo;

            if (di != null)
            {
                foreach (var dirInfo in di.GetFileSystemInfos())
                {
                    deleteFileSystemInfo(dirInfo);
                }
            }
            fsi.Delete();
        }
```

<p>6. 计算文件大小。</p>

```c#
		public void CalculateSize()
        {
            if(Directory.Exists(Path))
            {
                double result = 0;
                foreach (FileInfo file in new DirectoryInfo(Path).GetFiles("*", SearchOption.AllDirectories))
                    result = result + file.Length;
                Size = result;
            }
            else if (File.Exists(Path))
            {
                Size = new FileInfo(Path).Length;
            }
        }
```
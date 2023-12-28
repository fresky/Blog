---
layout: post
title: "遇到sql server的问题时如何排查"
date: 2013-06-27
comments: true
tags: Debug
---
<p><a href="http://www.i-programmer.info/programming/database/6028-the-first-things-i-look-at-on-a-sql-server-part-1.html">The First Things I Look At On A SQL Server</a>和<a href="http://www.i-programmer.info/programming/database/6028-the-first-things-i-look-at-on-a-sql-server-part-1.html?start=1">Page2</a>介绍了遇到sql server的问题时如何排查问题，<a href="http://www.i-programmer.info/images/stories/Core/Database/InitialChecks/dba_InitialChecks_part1%20%281%29.sql.html" target="_blank">Display Code</a>列出了sql代码。</p><p>包括如下10个检查：</p><ol><li>时间</li><li>sqlsever的版本</li><li>server instance的属性信息</li><li>操作系统信息</li><li>一些配置</li><li>做常见的waits</li><li>signal waits-CPU压力</li><li>sys.databases</li><li>database的CPU使用情况</li><li>database的内存使用情况</li></ol>